<template>
  <div class="container">
    <h1>検索</h1>
    <Modal :modalVideo="modalVideo" v-show="isModalShow" @closeModal="closeModal"/>
    <SearchChannel v-show="isSearchChannelShow" @closeSearchChannel="closeSearchChannel" @setChannelId="setChannelId"/>
    <div class="sort-search-bar">
      <button class="search-toggle-button" @click="toggleSearchList">検索する
          <i class="fas fa-chevron-down" :class="{rotate: isRotate}"></i>
      </button>
      <div class="divider"></div>
      <button
        v-for="sort in sortButtons"
        :key="sort.id"
        @click="changeSortMode(sort)"
        :class="{selected: sort.selected}"
      >
        {{ sort.title }}
      </button>
    </div>
    <div class="search-list">
        <div class="search-genre-inputs">
            <input type="text" v-model="selectedParams.q" placeholder="検索語句">
            <!-- <input type="text" v-model="selectedParams.maxResults" placeholder="表示件数(最大50件)"> -->
        </div>
        <div class="date-pickers">
            <DatePicker :language="ja" v-model="selectedParams.publishedAfter" class="date-picker" format="yyyy-MM-dd"/>
            <p>〜</p>
            <DatePicker :language="ja" v-model="selectedParams.publishedBefore" class="date-picker" format="yyyy-MM-dd"/>
        </div>
        <div class="search-channel">
            <button @click="openSearchChannel">チャンネル指定</button>
            <p>{{ selectedChannelTitle }}</p>
        </div>
        <div class="search-button">
            <button @click="fetchVideo">検索</button>
        </div>
        <div class="search-genre-buttons" v-for="param in searchParams" :key="param.name">
          <p>{{ param.title }}</p>
          <button v-for="item in param.items" :key="item.value" @click="changeParam(param, item)" :class="{selected: item.selected}">{{ item.label }}</button>
        </div>
    </div>
    <transition-group name="flip-list" tag="ul" :class="{toggled: isToggled}">
      <li v-for="video in videos" :key="video.id">
        <img :src="video.snippet.thumbnails.medium.url" alt="">
        <div class="duration-box">
            <p class="duration">{{ video.contentDetails.duration }}</p>
        </div>
        <p class="title">{{ video.snippet.omittedTitle }}</p>
        <p class="statistics">{{ video.statistics.viewCount}}回視聴・{{ video.statistics.likeCount }}Like・{{ video.statistics.commentCount }}コメント</p>
        <div class="hover-button">
          <button @click="openModal(video)">詳細</button>
          <button @click="openLink(video.id)">視聴</button>
        </div>
      </li>
    </transition-group>
  </div>
</template>
<script>
import Modal from '~/components/Modal.vue'
import SearchChannel from '~/components/SearchChannel.vue'
import DatePicker from 'vuejs-datepicker/src/components/Datepicker.vue'
import {ja} from 'vuejs-datepicker/dist/locale'
export default({
  components:{
    Modal,
    SearchChannel,
    DatePicker,
    ja
  },
  data() {
    return {
      microCMSKey: '',
      ja: ja,
      modalVideo: {
        "kind": "",
        "etag": "",
        "id": "",
        "statistics": {
            "viewCount": "",
            "likeCount": "",
            "favoriteCount": "",
            "commentCount": ""
        },
        "snippet": {
            "publishedAt": "",
            "channelId": "",
            "title": "",
            "description": "",
            "thumbnails": {
                "default": {
                    "url": "",
                    "width": "",
                    "height": ""
                },
                "medium": {
                    "url": "",
                    "width": "",
                    "height": ""
                },
                "high": {
                    "url": "",
                    "width": "",
                    "height": ""
                }
            },
            "channelTitle": "",
            "liveBroadcastContent": "",
            "publishTime": "",
            "omittedTitle": ""
        },
        "videoId": "",
        "contentDetails": {
            "duration": "",
            "dimension": "",
            "definition": "",
            "caption": "",
            "licensedContent": "",
            "contentRating": {},
            "projection": ""
        }
    },
      isModalShow: false,
      isToggled: true,
      isRotate: false,
      isSearchChannelShow: false,
      searchParams: [
          {
              title:'並び替え',
              name: 'order',
              items: [
                  {label:'関連度順',value: 'relevance',selected:true},
                  {label:'再生回数順',value: 'viewCount',selected:false},
                  {label:'アップロード順',value: 'date',selected:false},
                  {label:'高評価順',value: 'rating',selected:false}
              ]
          },
          {
              title:'検索タイプ',
              name: 'eventType',
              items: [
                  {label:'全て',value: '',selected:true},
                  {label:'進行中のライブ',value: 'live',selected:false},
                  {label:'既に終了したライブ',value: 'completed',selected:false},
                  {label:'今後配信予定のライブ',value: 'upcoming',selected:false}
              ]
          },
          {
              title:'字幕',
              name: 'videoCaption',
              items: [
                  {label:'全て',value: 'any',selected:true},
                  {label:'字幕あり', value: 'closedCaption',selected:false},
                  {label:'字幕なし', value: 'none',selected:false}
              ]
          },
          {
              title:'次元',
              name: 'videoDimension',
              items: [
                  {label:'全て',value: 'any',selected:true},
                  {label:'２D', value: '2d',selected:false},
                  {label:'３D', value: '3d',selected:false}
              ]
          },
          {
              title:'動画時間',
              name: 'videoDuration',
              items: [
                  {label:'全て',value: 'any',selected:true},
                  {label:'20分以上',value: 'long',selected:false},
                  {label:'4分以上20分未満',value: 'medium',selected:false},
                  {label:'4分未満',value: 'short',selected:false}
              ]
          }
    ],
    selectedParams:{
        part: 'snippet',
        key: '',
        q: '',
        maxResults: '50',
        order: 'relevance',
        type: 'video',
        eventType: '',
        videoCaption: 'any',
        videoDimension: 'any',
        videoDuration: 'any',
        channelId: '',
        publishedAfter: '',
        publishedBefore: ''
    },
    statisticsParams:{

    },
    selectedChannelTitle:'',
      videoIds:[],
      videos: [{
        "kind": "youtube#video",
        "etag": "5orQYjqkfYVDmDGNNjdWJTUSdDs",
        "id": "x1C0FD1XmTk",
        "statistics": {
            "viewCount": "35276",
            "likeCount": "2080",
            "favoriteCount": "0",
            "commentCount": "405"
        },
        "snippet": {
            "publishedAt": "2022-01-11T09:45:05Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "た",
            "description": "今回から過去最長シリーズの「た」が始まります！当チャンネルが始まって以来最大の難問に２人はどう立ち向かっていくのでしょうか！",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/x1C0FD1XmTk/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/x1C0FD1XmTk/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/x1C0FD1XmTk/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2022-01-11T09:45:05Z",
            "omittedTitle": "た"
        },
        "videoId": "x1C0FD1XmTk",
        "contentDetails": {
            "duration": "20:02",
            "dimension": "2d",
            "definition": "sd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "0ABaWqaTVH33VqSfly3Tbr8tr8U",
        "id": "wlQrQVzdoVA",
        "statistics": {
            "viewCount": "56641",
            "likeCount": "1841",
            "favoriteCount": "0",
            "commentCount": "404"
        },
        "snippet": {
            "publishedAt": "2022-01-08T00:45:03Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "大嘘つきに使える悪口「神聖ローマ帝国じゃん」【インテリ悪口パビリオン】#88",
            "description": "今回は大好評発売中の堀元の処女作『教養悪口本』にあやかってインテリ悪口パビリオンを開催（？）しました！このラジオを始めてから水野 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/wlQrQVzdoVA/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/wlQrQVzdoVA/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/wlQrQVzdoVA/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2022-01-08T00:45:03Z",
            "omittedTitle": "大嘘つきに使える悪口「神聖ローマ帝国じゃん」【インテリ悪口パ..."
        },
        "videoId": "wlQrQVzdoVA",
        "contentDetails": {
            "duration": "36:28",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "Uhei7JLEcCAN7IlZGuSb4D6cZgM",
        "id": "e4fDwDNc11Q",
        "statistics": {
            "viewCount": "61682",
            "likeCount": "1726",
            "favoriteCount": "0",
            "commentCount": "333"
        },
        "snippet": {
            "publishedAt": "2022-01-04T09:45:03Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "珍しい名字からは日本語の○○が分かる【うんちくエウレーカクイズ2】#87",
            "description": "以前視聴者から募集したうんちくクイズがついに動画化！皆さんやたらとハイレベルな問題を送ってきてくれているのですが、はたして全問 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/e4fDwDNc11Q/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/e4fDwDNc11Q/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/e4fDwDNc11Q/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2022-01-04T09:45:03Z",
            "omittedTitle": "珍しい名字からは日本語の○○が分かる【うんちくエウレーカクイ..."
        },
        "videoId": "e4fDwDNc11Q",
        "contentDetails": {
            "duration": "51:19",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "nyOgXyjSgLrn7PxnKmAYV5PLqsc",
        "id": "hyHkEbZDWmo",
        "statistics": {
            "viewCount": "50820",
            "likeCount": "1442",
            "favoriteCount": "0",
            "commentCount": "350"
        },
        "snippet": {
            "publishedAt": "2022-01-01T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "新年だから今後の構想をデカく語る【飛躍の年はクリシェ】【新年雑談回】#86",
            "description": "明けましておめでとうございます！本日は新年らしく今年の抱負を語りました。 ゆるい雑談回かと思いきや、超絶本気の告知がラストに ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/hyHkEbZDWmo/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/hyHkEbZDWmo/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/hyHkEbZDWmo/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2022-01-01T00:45:01Z",
            "omittedTitle": "新年だから今後の構想をデカく語る【飛躍の年はクリシェ】【新年..."
        },
        "videoId": "hyHkEbZDWmo",
        "contentDetails": {
            "duration": "45:41",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "e73qshHct_c7JFMf1t92muKpkZ0",
        "id": "poT4BzX7e_Q",
        "statistics": {
            "viewCount": "101762",
            "likeCount": "2230",
            "favoriteCount": "0",
            "commentCount": "152"
        },
        "snippet": {
            "publishedAt": "2021-12-28T14:53:54Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "ゆる言語学ラジオ忘年会ライブ【流行語大賞決定&amp;ひたすらエモい話】",
            "description": "ゆる言語学ラジオの忘年会ライブです。10万人突破記念ライブも兼ねてるよ！！絶対皆リアルタイムでコメントしてくれな！！！ 告知動画 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/poT4BzX7e_Q/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/poT4BzX7e_Q/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/poT4BzX7e_Q/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-28T14:53:54Z",
            "omittedTitle": "ゆる言語学ラジオ忘年会ライブ【流行語大賞決定&amp;ひたす..."
        },
        "videoId": "poT4BzX7e_Q",
        "contentDetails": {
            "duration": "3:37:43",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "KwYJJuiJn2zgKKnE-L-svFLRHpQ",
        "id": "Z0KLBPiRrOY",
        "statistics": {
            "viewCount": "81242",
            "likeCount": "2163",
            "favoriteCount": "0",
            "commentCount": "441"
        },
        "snippet": {
            "publishedAt": "2021-12-25T01:57:04Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "人は無知の量を誇るべき【雑談回】#85",
            "description": "雑談回。勉強すればするほど「オレ、これを知らないんだな」と自覚するし疑問の量が増えるので、むしろ無知の量で競うべきなのではないか ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/Z0KLBPiRrOY/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/Z0KLBPiRrOY/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/Z0KLBPiRrOY/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-25T01:57:04Z",
            "omittedTitle": "人は無知の量を誇るべき【雑談回】#85"
        },
        "videoId": "Z0KLBPiRrOY",
        "contentDetails": {
            "duration": "1:03:24",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "7SnD28YM-Zi5J_CqudYWt9brQu0",
        "id": "4jcgyHsqBOs",
        "statistics": {
            "viewCount": "49097",
            "likeCount": "1225",
            "favoriteCount": "0",
            "commentCount": "513"
        },
        "snippet": {
            "publishedAt": "2021-12-21T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "アジに「アジ」の名はふさわしくない【無限語源トーク2】#84",
            "description": "前回に引き続いて無限語源トークの2回目。水野曰く魚たちはそれぞれの特徴から名付けられているらしいのですが、アジは「味が良い」から ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/4jcgyHsqBOs/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/4jcgyHsqBOs/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/4jcgyHsqBOs/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-21T09:45:00Z",
            "omittedTitle": "アジに「アジ」の名はふさわしくない【無限語源トーク2】#84"
        },
        "videoId": "4jcgyHsqBOs",
        "contentDetails": {
            "duration": "22:20",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "RiPXyqCiP1LJGQaDXRFJykcjgGU",
        "id": "2UXylDl-HIY",
        "statistics": {
            "viewCount": "59597",
            "likeCount": "1451",
            "favoriteCount": "0",
            "commentCount": "418"
        },
        "snippet": {
            "publishedAt": "2021-12-18T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "『満月の夜なら』は、語源辞典から作詞された歌【無限語源トーク1】#83",
            "description": "今回は「無限語源トーク」の1回目。水野が延々と単語の語源を言うなか、隣の深読みおじさんが「あいみょん ＝ 語源オタク説」を唱えだしたり ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/2UXylDl-HIY/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/2UXylDl-HIY/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/2UXylDl-HIY/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-18T00:45:00Z",
            "omittedTitle": "『満月の夜なら』は、語源辞典から作詞された歌【無限語源トーク..."
        },
        "videoId": "2UXylDl-HIY",
        "contentDetails": {
            "duration": "25:54",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "tcFg3NXWOaNrNWjUffGiounfCjk",
        "id": "f4grx-2ngzE",
        "statistics": {
            "viewCount": "37683",
            "likeCount": "1210",
            "favoriteCount": "0",
            "commentCount": "305"
        },
        "snippet": {
            "publishedAt": "2021-12-14T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "【投票お願い】あなたの1票が我々の未来を変えます#82",
            "description": "今年もこの季節がやってきました。そう、JAPAN PODCAST AWARDSの季節です！どうしてもリスナーズチョイスの賞が欲しいので、 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/f4grx-2ngzE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/f4grx-2ngzE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/f4grx-2ngzE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-14T09:45:01Z",
            "omittedTitle": "【投票お願い】あなたの1票が我々の未来を変えます#82"
        },
        "videoId": "f4grx-2ngzE",
        "contentDetails": {
            "duration": "19:51",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "955Su_xOlD0i9wFtJIAYXt8NCpo",
        "id": "75HsFDb3HLI",
        "statistics": {
            "viewCount": "66646",
            "likeCount": "2104",
            "favoriteCount": "0",
            "commentCount": "268"
        },
        "snippet": {
            "publishedAt": "2021-12-11T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "日常系萌えアニメに潜む言語学の仮説【福田先生雑談回2】#81",
            "description": "前回に引き続き、今回も福田先生ゲスト雑談回！今回は言語学者になった理由や高校生時代の話など福田先生自身のお話を多く伺って ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/75HsFDb3HLI/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/75HsFDb3HLI/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/75HsFDb3HLI/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-11T00:45:00Z",
            "omittedTitle": "日常系萌えアニメに潜む言語学の仮説【福田先生雑談回2】#81"
        },
        "videoId": "75HsFDb3HLI",
        "contentDetails": {
            "duration": "41:52",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "_vrSaJ7cGBUEBbklHoXgat4qhGo",
        "id": "sSvxP5cUASM",
        "statistics": {
            "viewCount": "80079",
            "likeCount": "2304",
            "favoriteCount": "0",
            "commentCount": "296"
        },
        "snippet": {
            "publishedAt": "2021-12-07T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "言語学者が手加減せずに喋るとこうなる【福田先生雑談回1】#80",
            "description": "まだまだ福田先生ゲスト回は終わりません！今回は雑談回ということで水野が暴走気味に福田先生を言語学分野で質問攻めにしています。",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/sSvxP5cUASM/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/sSvxP5cUASM/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/sSvxP5cUASM/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-07T09:45:01Z",
            "omittedTitle": "言語学者が手加減せずに喋るとこうなる【福田先生雑談回1】#8..."
        },
        "videoId": "sSvxP5cUASM",
        "contentDetails": {
            "duration": "51:09",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "S4fYoU4B_gGSU-UHaFJklmPAnHI",
        "id": "2iwZmLJ5OnE",
        "statistics": {
            "viewCount": "44010",
            "likeCount": "1183",
            "favoriteCount": "0",
            "commentCount": "204"
        },
        "snippet": {
            "publishedAt": "2021-12-04T00:48:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "【忘年会ライブ告知】流行語大賞とか、サンプル1の出会いの話とか #79",
            "description": "今回は忘年会ライブの告知です。年末のお祭り騒ぎに乗じて、「ゆる言語学ラジオの流行語大賞」「2人の出会いなど、エモめの話」を語り ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/2iwZmLJ5OnE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/2iwZmLJ5OnE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/2iwZmLJ5OnE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-12-04T00:48:00Z",
            "omittedTitle": "【忘年会ライブ告知】流行語大賞とか、サンプル1の出会いの話と..."
        },
        "videoId": "2iwZmLJ5OnE",
        "contentDetails": {
            "duration": "25:33",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "9mOIuzDXUgh9-1xLjfhwOUvEBJM",
        "id": "0nmVZ6Up__k",
        "statistics": {
            "viewCount": "83064",
            "likeCount": "3053",
            "favoriteCount": "0",
            "commentCount": "551"
        },
        "snippet": {
            "publishedAt": "2021-11-30T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "問題文の言語によって正答率が左右される。そんなことある？【第二言語習得論5】#78",
            "description": "第二言語習得論ラスト回。「アラビア語話者は数直線が逆になる」「スペイン語だと解けるのにスウェーデン語だと解けない問題」「自販機で ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/0nmVZ6Up__k/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/0nmVZ6Up__k/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/0nmVZ6Up__k/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-30T09:45:00Z",
            "omittedTitle": "問題文の言語によって正答率が左右される。そんなことある？【第..."
        },
        "videoId": "0nmVZ6Up__k",
        "contentDetails": {
            "duration": "50:17",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "inzr3KB2pYlvcnKMGw8YzNiNufE",
        "id": "SmH9EbH0x0c",
        "statistics": {
            "viewCount": "67747",
            "likeCount": "1806",
            "favoriteCount": "0",
            "commentCount": "374"
        },
        "snippet": {
            "publishedAt": "2021-11-27T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "英語は衛星枠付け言語だった！？【第二言語習得論4】#77",
            "description": "今回は福田先生ゲスト回の2回目。明かされる衝撃の事実、なんと英語はあの衛星枠付言語だったのです。 【目次】 00:00 オープニング。",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/SmH9EbH0x0c/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/SmH9EbH0x0c/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/SmH9EbH0x0c/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-27T00:45:00Z",
            "omittedTitle": "英語は衛星枠付け言語だった！？【第二言語習得論4】#77"
        },
        "videoId": "SmH9EbH0x0c",
        "contentDetails": {
            "duration": "31:32",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "5MDanod2jsBFC9_VfhslLLMI4tU",
        "id": "4oKTEuDgO3s",
        "statistics": {
            "viewCount": "118020",
            "likeCount": "3481",
            "favoriteCount": "0",
            "commentCount": "413"
        },
        "snippet": {
            "publishedAt": "2021-11-23T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「無意識の学習」を証明する実験とは？【第二言語習得論3】#76",
            "description": "ついにこのチャンネルに言語学者が降臨しました。いつもよりも更に濃い内容となっておりますが、2人とは少し違ったガチの学者ならではノリ ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/4oKTEuDgO3s/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/4oKTEuDgO3s/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/4oKTEuDgO3s/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-23T09:45:00Z",
            "omittedTitle": "「無意識の学習」を証明する実験とは？【第二言語習得論3】#7..."
        },
        "videoId": "4oKTEuDgO3s",
        "contentDetails": {
            "duration": "38:43",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "HTz5a1FkGDsrYyVVw_Wn_KQoI1k",
        "id": "h2tt1bEU72g",
        "statistics": {
            "viewCount": "66137",
            "likeCount": "2026",
            "favoriteCount": "0",
            "commentCount": "421"
        },
        "snippet": {
            "publishedAt": "2021-11-20T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "母語はどこまで人に影響を与えるのか？方向感覚は？【第二言語習得論2】#75",
            "description": "左右を持たず、東西南北でのみ位置を示す言語があるそうです。それを使う民族はその分我々よりも位置感覚が優れていたりするのでしょ ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/h2tt1bEU72g/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/h2tt1bEU72g/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/h2tt1bEU72g/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-20T00:45:01Z",
            "omittedTitle": "母語はどこまで人に影響を与えるのか？方向感覚は？【第二言語習..."
        },
        "videoId": "h2tt1bEU72g",
        "contentDetails": {
            "duration": "16:50",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "2RmKsDG1S_8Ior-Y8IJ-Qvs1Tss",
        "id": "o3Yy_pjpBO8",
        "statistics": {
            "viewCount": "66589",
            "likeCount": "1974",
            "favoriteCount": "0",
            "commentCount": "312"
        },
        "snippet": {
            "publishedAt": "2021-11-16T09:45:02Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "明日から全く役に立たない第二言語習得論【第二言語習得論1】#74",
            "description": "今回のテーマは「第二言語習得論」。第二言語を習得する時の脳内について扱う学問ジャンルだそうです。すごく役立つ話が聞けそうな ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/o3Yy_pjpBO8/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/o3Yy_pjpBO8/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/o3Yy_pjpBO8/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-16T09:45:02Z",
            "omittedTitle": "明日から全く役に立たない第二言語習得論【第二言語習得論1】#..."
        },
        "videoId": "o3Yy_pjpBO8",
        "contentDetails": {
            "duration": "22:16",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "ZKcuXqmGjTQ5-Ml9X59-o_NXkv8",
        "id": "tu3kLecDqq4",
        "statistics": {
            "viewCount": "54986",
            "likeCount": "1567",
            "favoriteCount": "0",
            "commentCount": "338"
        },
        "snippet": {
            "publishedAt": "2021-11-14T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "サポーターコミュニティ始めます【課金で伝説のボツ回が見れる】 #73",
            "description": "「サポーターコミュニティ始めるよ！」という告知回です。 お気持ちで投げ銭をいただければ、伝説のボツ回「水野が難読漢字を書き取りする ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/tu3kLecDqq4/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/tu3kLecDqq4/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/tu3kLecDqq4/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-14T00:45:00Z",
            "omittedTitle": "サポーターコミュニティ始めます【課金で伝説のボツ回が見れる】..."
        },
        "videoId": "tu3kLecDqq4",
        "contentDetails": {
            "duration": "54:42",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "hBlLMMCHRY6czEaL8bFUIRFBBkI",
        "id": "CX-57sNSZeE",
        "statistics": {
            "viewCount": "81094",
            "likeCount": "2390",
            "favoriteCount": "0",
            "commentCount": "497"
        },
        "snippet": {
            "publishedAt": "2021-11-09T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "与謝野晶子に学ぶ、最強の黒歴史の作り方【奴隷合宿】#72",
            "description": "温泉旅館に来ました。浮かれてなぜか冒頭にVlogを挟みつつ、途中からいつものラジオになります。与謝野晶子はトルストイを読んで○○に ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/CX-57sNSZeE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/CX-57sNSZeE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/CX-57sNSZeE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-09T09:45:00Z",
            "omittedTitle": "与謝野晶子に学ぶ、最強の黒歴史の作り方【奴隷合宿】#72"
        },
        "videoId": "CX-57sNSZeE",
        "contentDetails": {
            "duration": "47:28",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "n4S_iBC5R6yRoDTSZsBlL4xW6oo",
        "id": "sj7eer2tArs",
        "statistics": {
            "viewCount": "104756",
            "likeCount": "2556",
            "favoriteCount": "0",
            "commentCount": "896"
        },
        "snippet": {
            "publishedAt": "2021-11-06T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "意図せずメタ認知が暴走する悲しき怪物【ミーム提案委員会2】＃71",
            "description": "大人気企画（？）のミーム提案委員会が帰ってきました。 今回は共感必至のあるあるミームからリスナーから送られてきたミームまで幅広く ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/sj7eer2tArs/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/sj7eer2tArs/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/sj7eer2tArs/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-06T00:45:01Z",
            "omittedTitle": "意図せずメタ認知が暴走する悲しき怪物【ミーム提案委員会2】＃..."
        },
        "videoId": "sj7eer2tArs",
        "contentDetails": {
            "duration": "54:37",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "nVwy35Jzk5oGnIt_XsruLWucitA",
        "id": "-d742iuB7L0",
        "statistics": {
            "viewCount": "87752",
            "likeCount": "2052",
            "favoriteCount": "0",
            "commentCount": "394"
        },
        "snippet": {
            "publishedAt": "2021-11-02T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "説教おじさんスイッチが反応しちゃう英単語【OEDおもしろ単語3】#70",
            "description": "おもしろ単語シリーズ、ラストです。今回のテーマは「悪口」。 「匿名の三流作家」「持説に惚れ込んでいる人」「出しゃばりな批評家」など、 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/-d742iuB7L0/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/-d742iuB7L0/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/-d742iuB7L0/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-11-02T09:45:00Z",
            "omittedTitle": "説教おじさんスイッチが反応しちゃう英単語【OEDおもしろ単語..."
        },
        "videoId": "-d742iuB7L0",
        "contentDetails": {
            "duration": "47:44",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "JvDEooQs3PbMrkC6PnHb3jiZcxg",
        "id": "WffHr9ypGsw",
        "statistics": {
            "viewCount": "88026",
            "likeCount": "2268",
            "favoriteCount": "0",
            "commentCount": "613"
        },
        "snippet": {
            "publishedAt": "2021-10-30T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "ジャルジャルのコントは1単語で表せる【OEDおもしろ単語2】#69",
            "description": "前回に続き、「おもしろ単語」紹介シリーズ。痴漢の話をしたくなる単語やサーフィンのコスパの悪さを語りたくなる単語など、色々出てくるよ！",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/WffHr9ypGsw/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/WffHr9ypGsw/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/WffHr9ypGsw/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-30T00:45:00Z",
            "omittedTitle": "ジャルジャルのコントは1単語で表せる【OEDおもしろ単語2】..."
        },
        "videoId": "WffHr9ypGsw",
        "contentDetails": {
            "duration": "39:57",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "vp2tdk3l964Ic_v4GRZSoj2KWdg",
        "id": "b5-G9dzdLzI",
        "statistics": {
            "viewCount": "191319",
            "likeCount": "5288",
            "favoriteCount": "0",
            "commentCount": "691"
        },
        "snippet": {
            "publishedAt": "2021-10-28T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "1年間辞書を読み続けた人にしか分からないあるある【OEDおもしろ単語1】#68",
            "description": "大型辞書を丸一年かけて全部読んだ人の本」を読んで、おもしろかった場所をまとめました。全然理解できないあるあるや、一生使わない ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/b5-G9dzdLzI/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/b5-G9dzdLzI/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/b5-G9dzdLzI/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-28T09:45:01Z",
            "omittedTitle": "1年間辞書を読み続けた人にしか分からないあるある【OEDおも..."
        },
        "videoId": "b5-G9dzdLzI",
        "contentDetails": {
            "duration": "35:20",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "LLp2g0QnoQYgNQGwRNiWfbXMhxo",
        "id": "Fc8ugpF5_C8",
        "statistics": {
            "viewCount": "122339",
            "likeCount": "2581",
            "favoriteCount": "0",
            "commentCount": "1405"
        },
        "snippet": {
            "publishedAt": "2021-10-26T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「ギガが減る」を許せない頑固おじさんの改心【今年の新語予想】#67",
            "description": "今回は三省堂の今年の新語2021に便乗してみました。実際に選ばれそうなものから絶対に選ばれそうにないものまで、いろいろな案が出た ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/Fc8ugpF5_C8/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/Fc8ugpF5_C8/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/Fc8ugpF5_C8/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-26T00:45:00Z",
            "omittedTitle": "「ギガが減る」を許せない頑固おじさんの改心【今年の新語予想】..."
        },
        "videoId": "Fc8ugpF5_C8",
        "contentDetails": {
            "duration": "1:04:31",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "Ra0CiYCGgcJnwo1KNghQMZ1irug",
        "id": "ru1ZVmytMoo",
        "statistics": {
            "viewCount": "84717",
            "likeCount": "2150",
            "favoriteCount": "0",
            "commentCount": "750"
        },
        "snippet": {
            "publishedAt": "2021-10-23T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "【徹底討論】プログラミング言語は言語なの？【ゆるコンピュータ科学ラジオ4】#66",
            "description": "ゆるコンピュータ科学ラジオ最終回。これまで見てきたプログラミング言語に関する解説をフル活用して「プログラミング言語は言語なのか？",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/ru1ZVmytMoo/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/ru1ZVmytMoo/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/ru1ZVmytMoo/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-23T00:45:00Z",
            "omittedTitle": "【徹底討論】プログラミング言語は言語なの？【ゆるコンピュータ..."
        },
        "videoId": "ru1ZVmytMoo",
        "contentDetails": {
            "duration": "59:17",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "GpE2L7d8HvD_etniGcDyLn6bo4c",
        "id": "qNHfKNjX8Us",
        "statistics": {
            "viewCount": "83830",
            "likeCount": "2491",
            "favoriteCount": "0",
            "commentCount": "608"
        },
        "snippet": {
            "publishedAt": "2021-10-19T09:45:02Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "プログラミング言語には思想が宿る。だから戦争が起きる。【ゆるコンピュータ科学ラジオ3】#65",
            "description": "ゆるコンピュータ科学ラジオも早3回目。今回はプログラミング言語に宿る思想について見ていきます。プログラミング言語も思想によって進化 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/qNHfKNjX8Us/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/qNHfKNjX8Us/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/qNHfKNjX8Us/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-19T09:45:02Z",
            "omittedTitle": "プログラミング言語には思想が宿る。だから戦争が起きる。【ゆる..."
        },
        "videoId": "qNHfKNjX8Us",
        "contentDetails": {
            "duration": "50:16",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "XnoWTOjjNaAuHugoh0NMnSF5RMo",
        "id": "uDCTXGCk2Zk",
        "statistics": {
            "viewCount": "80641",
            "likeCount": "2603",
            "favoriteCount": "0",
            "commentCount": "479"
        },
        "snippet": {
            "publishedAt": "2021-10-16T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "プログラマーと辞書オタク、実質同じ【ゆるコンピュータ科学ラジオ2】#64",
            "description": "プログラマーとは何をしている人たちなのでしょうか？彼らが普段パソコンに向かって行っている作業が何なのかを知れば、自ずと ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/uDCTXGCk2Zk/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/uDCTXGCk2Zk/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/uDCTXGCk2Zk/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-16T00:45:01Z",
            "omittedTitle": "プログラマーと辞書オタク、実質同じ【ゆるコンピュータ科学ラジ..."
        },
        "videoId": "uDCTXGCk2Zk",
        "contentDetails": {
            "duration": "39:23",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "_jbR2ENOIhFU4Uewb0XrA8rc27k",
        "id": "dkP8Uf7PveE",
        "statistics": {
            "viewCount": "72735",
            "likeCount": "2673",
            "favoriteCount": "0",
            "commentCount": "527"
        },
        "snippet": {
            "publishedAt": "2021-10-12T09:45:04Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "脳にUSBを挿したらYouTube再生できる？【ゆるコンピュータ科学ラジオ1】#63",
            "description": "今回は「ゆるコンピュータ科学ラジオ」です。以前出た「プログラミング言語は言語か？」みたいな話をするために、プログラミング言語について ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/dkP8Uf7PveE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/dkP8Uf7PveE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/dkP8Uf7PveE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-12T09:45:04Z",
            "omittedTitle": "脳にUSBを挿したらYouTube再生できる？【ゆるコンピュ..."
        },
        "videoId": "dkP8Uf7PveE",
        "contentDetails": {
            "duration": "29:09",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "HGfbB4Uhj6TbquKTmA3nWcdtr2g",
        "id": "ugPrgVrR6ag",
        "statistics": {
            "viewCount": "54323",
            "likeCount": "1674",
            "favoriteCount": "0",
            "commentCount": "313"
        },
        "snippet": {
            "publishedAt": "2021-10-09T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "隣の棚はアンパンマンでした【文教堂フェア行ってきた】#62",
            "description": "文教堂で実施してくれた「ゆる言語学ラジオフェア」、実際に店舗に行って撮影してきました。フェアの様子をお伝えしつつ、「生成文法の本 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/ugPrgVrR6ag/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/ugPrgVrR6ag/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/ugPrgVrR6ag/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-09T00:45:00Z",
            "omittedTitle": "隣の棚はアンパンマンでした【文教堂フェア行ってきた】#62"
        },
        "videoId": "ugPrgVrR6ag",
        "contentDetails": {
            "duration": "24:19",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "y9e624y3jHFcPQMyZA09Us-oZjM",
        "id": "SbV9O7Gd4Sk",
        "statistics": {
            "viewCount": "73590",
            "likeCount": "1871",
            "favoriteCount": "0",
            "commentCount": "990"
        },
        "snippet": {
            "publishedAt": "2021-10-05T09:45:03Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "人類の多くはベンジャミン。生まれた瞬間〇〇を判断【英米人名２】#61",
            "description": "今回は前回に引き続き「英米人名語源」がテーマです。語源を知れば、あの名前とこの名前が同一のものだと分かったり、自分がベンジャミン ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/SbV9O7Gd4Sk/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/SbV9O7Gd4Sk/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/SbV9O7Gd4Sk/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-05T09:45:03Z",
            "omittedTitle": "人類の多くはベンジャミン。生まれた瞬間〇〇を判断【英米人名２..."
        },
        "videoId": "SbV9O7Gd4Sk",
        "contentDetails": {
            "duration": "44:31",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "tY55tArG2PWwxtRlgqFlN-m_EwE",
        "id": "bkZbSiwHBWc",
        "statistics": {
            "viewCount": "117377",
            "likeCount": "2563",
            "favoriteCount": "0",
            "commentCount": "617"
        },
        "snippet": {
            "publishedAt": "2021-10-02T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「許してクレメンス」は超インテリギャグ【英米人名1】#60",
            "description": "今回のテーマは「英米人名語源」。ウィリアムは「決意の兜」を意味するし、アタナシウスは「不死」を意味するそうです。鬼滅の刃に「不死川」 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/bkZbSiwHBWc/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/bkZbSiwHBWc/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/bkZbSiwHBWc/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-10-02T00:45:00Z",
            "omittedTitle": "「許してクレメンス」は超インテリギャグ【英米人名1】#60"
        },
        "videoId": "bkZbSiwHBWc",
        "contentDetails": {
            "duration": "34:40",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "ZfQVMOZ_SPH4p9DlUPLuMrvGYcE",
        "id": "EtXBKIMqSUY",
        "statistics": {
            "viewCount": "68250",
            "likeCount": "1690",
            "favoriteCount": "0",
            "commentCount": "823"
        },
        "snippet": {
            "publishedAt": "2021-09-28T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "米国を恐怖に陥れた「サメの夏」をミーム化【雑談コメント返し】 #59",
            "description": "今回は久しぶりのコメント返し回……のはずが、筒井康隆の短編の話が止まらなくなってしまっています。仕方がないのです。2人とも筒井 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/EtXBKIMqSUY/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/EtXBKIMqSUY/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/EtXBKIMqSUY/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-28T09:45:01Z",
            "omittedTitle": "米国を恐怖に陥れた「サメの夏」をミーム化【雑談コメント返し】..."
        },
        "videoId": "EtXBKIMqSUY",
        "contentDetails": {
            "duration": "58:26",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "6aqmOo2WoG8oxbc7fVowJs3dJ1o",
        "id": "T5cDcCKB19k",
        "statistics": {
            "viewCount": "78942",
            "likeCount": "1629",
            "favoriteCount": "0",
            "commentCount": "492"
        },
        "snippet": {
            "publishedAt": "2021-09-25T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "江戸時代の米はビットコインに似ている【雑談回】 #58",
            "description": "サピア撮り終わり雑談回。「太宰治が推奨してるクリエイティブの引き算」「江戸時代の米はビットコインのように価格が乱高下した」「富士そば ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/T5cDcCKB19k/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/T5cDcCKB19k/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/T5cDcCKB19k/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-25T00:45:01Z",
            "omittedTitle": "江戸時代の米はビットコインに似ている【雑談回】 #58"
        },
        "videoId": "T5cDcCKB19k",
        "contentDetails": {
            "duration": "1:04:07",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "2Y54rqhHcj47tecJmVfENa0HugU",
        "id": "fFbumZyreQA",
        "statistics": {
            "viewCount": "103871",
            "likeCount": "2248",
            "favoriteCount": "0",
            "commentCount": "576"
        },
        "snippet": {
            "publishedAt": "2021-09-21T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "子音が17個連続する言語がある！？『言語』よもやま話【サピア4】#57",
            "description": "言語の変化は5ステップで説明できる」「子音が17個連続する言語がある！？」など、サピアの著書の『言語』から面白そうなところを集めてき ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/fFbumZyreQA/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/fFbumZyreQA/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/fFbumZyreQA/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-21T00:45:01Z",
            "omittedTitle": "子音が17個連続する言語がある！？『言語』よもやま話【サピア..."
        },
        "videoId": "fFbumZyreQA",
        "contentDetails": {
            "duration": "55:08",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "XmX5Fphs74wJD6eVGzUem1nBfr4",
        "id": "HwuXR3KH0wI",
        "statistics": {
            "viewCount": "61248",
            "likeCount": "1832",
            "favoriteCount": "0",
            "commentCount": "441"
        },
        "snippet": {
            "publishedAt": "2021-09-18T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「ら抜き言葉」で日本語は美しくなった【サピア3】 #56",
            "description": "「けしからん！」と言われがちな「ら抜き言葉」ですが、実は美しい日本語への変化なのかもしれません。 エドワード・サピアの指摘した「 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/HwuXR3KH0wI/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/HwuXR3KH0wI/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/HwuXR3KH0wI/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-18T00:45:01Z",
            "omittedTitle": "「ら抜き言葉」で日本語は美しくなった【サピア3】 #56"
        },
        "videoId": "HwuXR3KH0wI",
        "contentDetails": {
            "duration": "19:41",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "hQGO8tx9-jb4t0I3AKPhgoPiZq4",
        "id": "h6zyDXsuVh8",
        "statistics": {
            "viewCount": "59759",
            "likeCount": "1426",
            "favoriteCount": "0",
            "commentCount": "365"
        },
        "snippet": {
            "publishedAt": "2021-09-14T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "言語の変化を説明する鍵は「ドリフト」【サピア2】#55",
            "description": "サピアを大いに悩ませた英文“Whom did you see”。文法的には正しいはずのこの文をネイティブは気持ち悪く感じてしまうそうです。 我々に ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/h6zyDXsuVh8/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/h6zyDXsuVh8/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/h6zyDXsuVh8/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-14T09:45:01Z",
            "omittedTitle": "言語の変化を説明する鍵は「ドリフト」【サピア2】#55"
        },
        "videoId": "h6zyDXsuVh8",
        "contentDetails": {
            "duration": "35:59",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "VNyHDoIrsA-h1Cg5fuSGzXsk-nM",
        "id": "purzZplAHpI",
        "statistics": {
            "viewCount": "96798",
            "likeCount": "2093",
            "favoriteCount": "0",
            "commentCount": "342"
        },
        "snippet": {
            "publishedAt": "2021-09-11T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "言語学の研究対象は、文字よりも音よりも○○【サピア1】#54",
            "description": "今回のテーマは「エドワード・サピア」。今まで扱ってきた言語学者達と比べても異質の存在です。彼の説いた「人間に音声器官は存在 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/purzZplAHpI/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/purzZplAHpI/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/purzZplAHpI/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-11T00:45:01Z",
            "omittedTitle": "言語学の研究対象は、文字よりも音よりも○○【サピア1】#54"
        },
        "videoId": "purzZplAHpI",
        "contentDetails": {
            "duration": "44:57",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "WT9uEIG9lsf6oAMF1jJMDC9L3ek",
        "id": "LteliiwAFe4",
        "statistics": {
            "viewCount": "56803",
            "likeCount": "1936",
            "favoriteCount": "0",
            "commentCount": "234"
        },
        "snippet": {
            "publishedAt": "2021-09-07T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "人類が服を着始めた年代は、あの虫から分かる【うんちくエウレーカクイズ】 #53",
            "description": "新コーナー「うんちくエウレーカクイズ」です。「ナチスに絵を売って英雄になるってどういうこと？」「国を戦争に負けさせる以上の重罪って？",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/LteliiwAFe4/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/LteliiwAFe4/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/LteliiwAFe4/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-07T09:45:01Z",
            "omittedTitle": "人類が服を着始めた年代は、あの虫から分かる【うんちくエウレー..."
        },
        "videoId": "LteliiwAFe4",
        "contentDetails": {
            "duration": "32:30",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "1EhHpWv17QrMt23u_OY5Po8usck",
        "id": "FLq-XlEvxak",
        "statistics": {
            "viewCount": "86103",
            "likeCount": "1953",
            "favoriteCount": "0",
            "commentCount": "1000"
        },
        "snippet": {
            "publishedAt": "2021-09-04T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "オタク用語「しんどい」の精神は古文で既に登場してる【雑談回】#52",
            "description": "オタク用語「しんどい」は枕草子にも登場してる・日常会話にエスケープ記号や型宣言を取り入れたい・パラメーターとバロメーターの違い。",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/FLq-XlEvxak/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/FLq-XlEvxak/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/FLq-XlEvxak/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-09-04T00:45:00Z",
            "omittedTitle": "オタク用語「しんどい」の精神は古文で既に登場してる【雑談回】..."
        },
        "videoId": "FLq-XlEvxak",
        "contentDetails": {
            "duration": "54:06",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "HCbluxo_1IToYVZEAL6GJ-2zqog",
        "id": "O9dMmofn7JU",
        "statistics": {
            "viewCount": "66200",
            "likeCount": "2291",
            "favoriteCount": "0",
            "commentCount": "259"
        },
        "snippet": {
            "publishedAt": "2021-08-31T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "妄想で人を撃ち、自分のアレを切り落とした狂人の皮肉【オックスフォード英語大辞典2】#51",
            "description": "「オックスフォード英語大辞典」シリーズの続き。今回はとうとう最大の協力者である殺人犯が登場します。「自分が殺した人の奥さんとの奇妙 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/O9dMmofn7JU/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/O9dMmofn7JU/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/O9dMmofn7JU/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-31T09:45:01Z",
            "omittedTitle": "妄想で人を撃ち、自分のアレを切り落とした狂人の皮肉【オックス..."
        },
        "videoId": "O9dMmofn7JU",
        "contentDetails": {
            "duration": "35:41",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "HuPklkicxIS0oYWodckDrZxsu2E",
        "id": "e11Q7m-45Cc",
        "statistics": {
            "viewCount": "90589",
            "likeCount": "2376",
            "favoriteCount": "0",
            "commentCount": "246"
        },
        "snippet": {
            "publishedAt": "2021-08-28T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "世界初の大型辞書は、殺人犯のお陰で完成した【オックスフォード英語大辞典1】#50",
            "description": "今回のテーマは「オックスフォード英語大辞典」。「熱心な協力者が殺人犯だった」「編集主幹は牛にラテン語を教えていた」「完成まで70年 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/e11Q7m-45Cc/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/e11Q7m-45Cc/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/e11Q7m-45Cc/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-28T00:45:00Z",
            "omittedTitle": "世界初の大型辞書は、殺人犯のお陰で完成した【オックスフォード..."
        },
        "videoId": "e11Q7m-45Cc",
        "contentDetails": {
            "duration": "34:07",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "wshKJ7ZL6_W4aP59hMnR7aCi5pM",
        "id": "7sX8rPt2uYE",
        "statistics": {
            "viewCount": "53661",
            "likeCount": "1597",
            "favoriteCount": "0",
            "commentCount": "294"
        },
        "snippet": {
            "publishedAt": "2021-08-24T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「お前の母ちゃんデベソ」の起源は御成敗式目【書店コラボ告知】 #49",
            "description": "文教堂あきる野とうきゅう店さんとコラボで、「ゆる言語学ラジオフェア」をやることになりました。皆さんぜひ足を運んでくださいね！&フェアに ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/7sX8rPt2uYE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/7sX8rPt2uYE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/7sX8rPt2uYE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-24T09:45:00Z",
            "omittedTitle": "「お前の母ちゃんデベソ」の起源は御成敗式目【書店コラボ告知】..."
        },
        "videoId": "7sX8rPt2uYE",
        "contentDetails": {
            "duration": "38:23",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "HHCOiG_7Yj6gi7ocxmIOQaDB80k",
        "id": "VNTx4A8C6qU",
        "statistics": {
            "viewCount": "63680",
            "likeCount": "1509",
            "favoriteCount": "0",
            "commentCount": "534"
        },
        "snippet": {
            "publishedAt": "2021-08-21T00:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "数と言葉はどちらも「身体ハック」から生まれた【数の発明3】#48",
            "description": "『数の発明』シリーズ最終回。「数と言葉は両方とも身体をハックして生まれた」「数がなければ文字もなかった？」「自分の理解度に応じた ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/VNTx4A8C6qU/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/VNTx4A8C6qU/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/VNTx4A8C6qU/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-21T00:45:00Z",
            "omittedTitle": "数と言葉はどちらも「身体ハック」から生まれた【数の発明3】#..."
        },
        "videoId": "VNTx4A8C6qU",
        "contentDetails": {
            "duration": "38:25",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "ZqHPIdh2vXHcNSZ_khE2c9Xa_to",
        "id": "Idn-gber9-A",
        "statistics": {
            "viewCount": "67129",
            "likeCount": "1741",
            "favoriteCount": "0",
            "commentCount": "575"
        },
        "snippet": {
            "publishedAt": "2021-08-17T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "10進法が生まれた究極の原因は「石川啄木」【数の発明2】#47",
            "description": "『数の発明』シリーズの続き。今回は「10進法はなぜ発明されたのか？」「12進数の方が便利じゃね？」「人類が巨大な数を必要とした ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/Idn-gber9-A/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/Idn-gber9-A/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/Idn-gber9-A/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-17T09:45:00Z",
            "omittedTitle": "10進法が生まれた究極の原因は「石川啄木」【数の発明2】#4..."
        },
        "videoId": "Idn-gber9-A",
        "contentDetails": {
            "duration": "37:27",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "y0T_e4X8lCe6LHRmO4qUWy84BnI",
        "id": "jrNc7fmtTNE",
        "statistics": {
            "viewCount": "72405",
            "likeCount": "2043",
            "favoriteCount": "0",
            "commentCount": "587"
        },
        "snippet": {
            "publishedAt": "2021-08-14T00:45:03Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "人は生まれつき算数ができる？赤ちゃんビビらす実験とは【数の発明1】#46",
            "description": "今回のテーマは『数の発明』。「人間は数を本能的に持っているのか？それとも文化的に習得するのか？」という疑問に取り組んだケイレブ・ ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/jrNc7fmtTNE/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/jrNc7fmtTNE/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/jrNc7fmtTNE/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-14T00:45:03Z",
            "omittedTitle": "人は生まれつき算数ができる？赤ちゃんビビらす実験とは【数の発..."
        },
        "videoId": "jrNc7fmtTNE",
        "contentDetails": {
            "duration": "28:16",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "CE2R0149QP2VqJ76VvpDIKFOQH0",
        "id": "o9xAhJ2ZbRQ",
        "statistics": {
            "viewCount": "137392",
            "likeCount": "2980",
            "favoriteCount": "0",
            "commentCount": "530"
        },
        "snippet": {
            "publishedAt": "2021-08-10T09:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "会話にキモインテリ慣用句を放り込め！【何こいつキモナイト】#45",
            "description": "新コーナー「何こいつキモナイト」です。「ケレスとバッカスがいないとヴィーナスが凍えてしまう」「ディオゲネスの樽」など、普通に使うとキモい ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/o9xAhJ2ZbRQ/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/o9xAhJ2ZbRQ/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/o9xAhJ2ZbRQ/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-10T09:45:01Z",
            "omittedTitle": "会話にキモインテリ慣用句を放り込め！【何こいつキモナイト】#..."
        },
        "videoId": "o9xAhJ2ZbRQ",
        "contentDetails": {
            "duration": "59:39",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "MjAH6Jnk67bmR_ip5d0F860rU1I",
        "id": "A1_ScH1NiCo",
        "statistics": {
            "viewCount": "115008",
            "likeCount": "3508",
            "favoriteCount": "0",
            "commentCount": "598"
        },
        "snippet": {
            "publishedAt": "2021-08-07T01:48:40Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "ネイティブは存在しない動詞も理解できるらしい…【カタルシス英文法_文型2】#44",
            "description": "前回に引き続き、「文型」です。「存在しない動詞も文型で理解できる」「あげる・もらう・妬むの共通点」「物理をマスターしたニュアンスがあるの ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/A1_ScH1NiCo/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/A1_ScH1NiCo/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/A1_ScH1NiCo/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-07T01:48:40Z",
            "omittedTitle": "ネイティブは存在しない動詞も理解できるらしい…【カタルシス英..."
        },
        "videoId": "A1_ScH1NiCo",
        "contentDetails": {
            "duration": "50:44",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "i4WFcreMWJOZiXN5eMIm_ltlZDI",
        "id": "FeSir-QJmUs",
        "statistics": {
            "viewCount": "70160",
            "likeCount": "2134",
            "favoriteCount": "0",
            "commentCount": "290"
        },
        "snippet": {
            "publishedAt": "2021-08-03T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "高校英語で習う「5文型」、実は超役に立つ【カタルシス英文法_文型1】#43",
            "description": "今回のテーマは「文型」。高校英語で習ったアレ、何の役にも立たないと思ってませんか？なんと、文型さえ分かれば動詞の意味が推測 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/FeSir-QJmUs/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/FeSir-QJmUs/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/FeSir-QJmUs/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-08-03T09:45:00Z",
            "omittedTitle": "高校英語で習う「5文型」、実は超役に立つ【カタルシス英文法_..."
        },
        "videoId": "FeSir-QJmUs",
        "contentDetails": {
            "duration": "30:46",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "2hG5rYhyX77vCRIVkF_7y0a87M8",
        "id": "kNIQXzBiTwA",
        "statistics": {
            "viewCount": "104697",
            "likeCount": "2094",
            "favoriteCount": "0",
            "commentCount": "707"
        },
        "snippet": {
            "publishedAt": "2021-07-31T00:45:01Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "「便」はなぜ「手紙」も「うんこ」も表すのか【雑談コメント返し】#42",
            "description": "雑談コメント返し回。「seasonはなぜ季節と味付けを表す？」「\"便\"はなぜ手紙とうんこを表す？」「Dr.Stoneは語源で見せ場を作ってる ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/kNIQXzBiTwA/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/kNIQXzBiTwA/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/kNIQXzBiTwA/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-07-31T00:45:01Z",
            "omittedTitle": "「便」はなぜ「手紙」も「うんこ」も表すのか【雑談コメント返し..."
        },
        "videoId": "kNIQXzBiTwA",
        "contentDetails": {
            "duration": "58:25",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    },
    {
        "kind": "youtube#video",
        "etag": "a3pKbrybEFj6lFiTnz7iCXaUlJQ",
        "id": "43bvI0smi7k",
        "statistics": {
            "viewCount": "44591",
            "likeCount": "1013",
            "favoriteCount": "0",
            "commentCount": "251"
        },
        "snippet": {
            "publishedAt": "2021-07-27T09:45:00Z",
            "channelId": "UCmpkIzF3xFzhPez7gXOyhVg",
            "title": "助数詞シリーズは『宇宙兄弟』っぽいよね（自画自賛）【振り返り雑談回】#41",
            "description": "長大な「助数詞」シリーズを撮り終えた達成感で、振り返り雑談回をやりました。「プルーストにはない週刊連載の強み」「ピダハンと喧嘩商売 ...",
            "thumbnails": {
                "default": {
                    "url": "https://i.ytimg.com/vi/43bvI0smi7k/default.jpg",
                    "width": 120,
                    "height": 90
                },
                "medium": {
                    "url": "https://i.ytimg.com/vi/43bvI0smi7k/mqdefault.jpg",
                    "width": 320,
                    "height": 180
                },
                "high": {
                    "url": "https://i.ytimg.com/vi/43bvI0smi7k/hqdefault.jpg",
                    "width": 480,
                    "height": 360
                }
            },
            "channelTitle": "ゆる言語学ラジオ",
            "liveBroadcastContent": "none",
            "publishTime": "2021-07-27T09:45:00Z",
            "omittedTitle": "助数詞シリーズは『宇宙兄弟』っぽいよね（自画自賛）【振り返り..."
        },
        "videoId": "43bvI0smi7k",
        "contentDetails": {
            "duration": "24:06",
            "dimension": "2d",
            "definition": "hd",
            "caption": "false",
            "licensedContent": true,
            "contentRating": {},
            "projection": "rectangular"
        }
    }
],
      sortButtons:[
        {title: '新しい順', standard1: 'snippet', standard2: 'publishedAt', selected:true, id: 1},
        {title: '高評価順', standard1: 'statistics', standard2: 'likeCount', selected:false, id: 2},
        {title: '視聴回数順', standard1: 'statistics', standard2: 'viewCount', selected:false, id: 3},
        {title: 'コメント数順', standard1: 'statistics', standard2: 'commentCount', selected:false, id: 4},
        {title: '動画時間順', standard1: 'contentDetails', standard2: 'duration', selected:false, id: 5},
        {title: '高評価率順', standard1: 'statistics', standard2: 'likeCount', standard3: 'statistics', standard4: 'viewCount', selected:false, id: 6},
        {title: 'コメント率順', standard1: 'statistics', standard2: 'commentCount', standard3: 'statistics', standard4: 'viewCount', selected:false, id: 7}
      ]
    }
  },
  methods:{
    async fetchVideo({ $axios }) {
      this.AllSortButtonOff()
      this.toggleSearchList()
      this.modifySelectedParams()
      const url = "https://www.googleapis.com/youtube/v3/"
      const that = this
    //   const key = that.keys[Math.floor(Math.random() * this.keys.length)]
      await this.$axios.$get
        (url + 'search', {
          params: this.selectedParams
        }).catch(function(error) {
          console.log('ERROR!')
        }).then(function(response) {
          that.videoIds = response.items
          that.videos = []
        })
      for (let i = 0; i < that.videoIds.length; i++) {
        that.videoIds[i].videoId = that.videoIds[i].id.videoId
        await this.$axios.$get
        (url + 'videos', {
          params:{
            part: 'statistics',
            key: that.microCMSKey,
            id: that.videoIds[i].id.videoId
          }
        }).catch(function(error) {
          console.log('ERROR!')
        }).then(function(response) {
          console.log(response)
          that.videoIds[i] = {...response.items[0], ...that.videoIds[i]}
        })
        console.log(that.videoIds)
        await this.$axios.$get
        (url + 'videos', {
          params:{
            part: 'contentDetails',
            key: that.microCMSKey,
            id: that.videoIds[i].id.videoId
          }
        }).catch(function(error) {
          console.log('ERROR!')
        }).then(function(response) {
          that.videoIds[i].snippet.omittedTitle = that.omitTitle(that.videoIds[i].snippet.title).split('&amp;').join('&')
          response.items[0].contentDetails.duration = that.modifyISO(response.items[0].contentDetails.duration)
          that.videos.push({...that.videoIds[i], ...response.items[0]})
        })
      }
    },
    omitTitle(title) {
        if(title.length > 30) {
          return title.substr(0, 30) + '...'
        }else{
          return title
        }
    },
    modifyISO(iso) {
      const Arr = iso.slice(2, -1).split(/H|M/)
      if(/^PT\d+H$/.test(iso)) {
          return Arr[0] + ":00:00"
      }else if(/^PT\d+M$/.test(iso)) {
          return Arr[0] + ":00"
      }else if(/^PT\d+H\d+S$/.test(iso)) {
          if(Arr[1].length === 1) {
             return Arr[0] + ":00:0" + Arr[1]
          }
          return Arr[0] + ":00:" + Arr[1]
      }else{
        for(let i=0;i<Arr.length;i++) {
            if(Arr[i].length === 1) {
            Arr[i] = "0" + Arr[i]
            }
        }
        if(Arr.length === 1){
            Arr[1] = Arr[0]
            Arr[0] = '0'
        }else if(Arr[0][0] === '0'){
            Arr[0] = Arr[0].substr(1, 1)
        }
        return Arr.join(":")
      }
    },
    modifySelectedParams() {
      for(let i in this.selectedParams) {
        if(!this.selectedParams[i]) {
            delete this.selectedParams[i]
        }
      }
    },
    changeSortMode(item) {
      if(item.standard3){
          this.sortVideo(item.standard1, item.standard2, item.standard3, item.standard4)
      }else{
          this.sortVideo(item.standard1, item.standard2)
      }
      this.changeSortButtonStyle(item)
    },
    sortVideo(s1, s2, s3, s4) {
      if(s3){
        this.videos.sort(function(a, b) {
            return b[s1][s2] / b[s3][s4] - a[s1][s2] / a[s3][s4]
          })
      }else{
        this.videos.sort(function(a, b) {
            if(s2 === 'publishedAt'){
            if(b[s1][s2] > a[s1][s2]) {
                return 1
            }else{
                return -1
            }
            }else if(s2 === 'duration'){
            return b[s1][s2].split(':').join('') - a[s1][s2].split(':').join('')
            }else{
            return b[s1][s2] - a[s1][s2]
        }
        })
      }
    },
    AllSortButtonOff(){
      this.sortButtons.some(function(sort) {
        sort.selected = false
      })
    },
    changeSortButtonStyle(item) {
      this.AllSortButtonOff()
      item.selected = true
    },
    openLink(id) {
      window.open('https://www.youtube.com/watch?v=' + id, '_blank')
    },
    openModal(video) {
      this.modalVideo = video
      this.isModalShow = true
    },
    closeModal() {
        this.isModalShow = false
    },
    openSearchChannel() {
        this.isSearchChannelShow = true
    },
    closeSearchChannel() {
        this.isSearchChannelShow = false
    },
    changeParam(param, item) {
      param.items.some(function(item) {
        item.selected = false
      })
      item.selected = true
      this.selectedParams[param.name] = item.value
    },
    toggleSearchList() {
        this.isToggled = !this.isToggled
        this.isRotate = !this.isRotate
    },
    setChannelId(data) {
        this.selectedParams.channelId = data.id.channelId
        this.selectedChannelTitle = data.snippet.title
    }
  },
  async mounted() {
      let that = this
      const key =process.env.MICROCMS_KEY
      await this.$axios.$get('https://typing.microcms.io/api/v1/youtube-sort-key', {
        headers: { 'X-MICROCMS-API-KEY': key}
      }).then(function(response) {
          that.microCMSKey = response.contents[0].key
          that.selectedParams.key = response.contents[0].key
      })
  }
})
</script>
<style scoped>
.container{
  width: 77%;
  max-width: calc(100% - 225px);
  padding: 0 4%;
  position: absolute;
  top: 0;
  left: 15%;
  overflow: scroll;
}
.sort-search-bar{
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.sort-search-bar button{
  margin: 0 5px;
  width: 15%;
  padding: 8px 0;
  border-radius: 8px;
  background-color: rgb(83, 87, 95);
  color: white;
  border: 1px solid rgba(153, 157, 165, 0.5);
  cursor: pointer;
  transition: .3s;
}
.sort-search-bar button:hover{
  background-color: rgb(103, 107, 115);
}
.search-toggle-button{
  position: relative;
  background-color: rgb(243, 130, 125) !important;
  border: none !important;
}
.fa-chevron-down{
  position: absolute;
  right: 10%;
  top: 50%;
  transform: translateY(-50%);
  transition: .5s;
}
.rotate{
  transform: translateY(-60%)rotateX(180deg);
}
.divider{
  width: 2px;
  height: 40px;
  margin: 0 8px;
  background: rgb(103, 107, 115);
}
.selected{
  background-color: white !important;
  color: rgb(53, 57, 65) !important;
  cursor: default !important;
}
.selected:hover{
  opacity: 1 !important;
}
.search-list{
  padding: 20px 0;
  height: auto;
  overflow: hidden;
  transition: transform .5s;
}
.search-genre-inputs{
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}
.search-list input{
    width: 50%;
    height: 30px;
    color: white;
    background: none;
    outline: none;
    border: none;
    border-bottom: 1px solid white;
    text-align: center;
    margin: 15px auto;
    font-size: 1.5em;
    transition: .5s;
}
.search-list input:focus{
    width: 40%;
}
.date-pickers{
    display: flex;
    justify-content: center;
    align-items: center;
}
.date-picker{
    color: rgb(53,57,65);
    margin: 0 20px;
}
.search-channel{
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    margin: 10px 0;
}
.search-channel button{
    width: 20%;
    margin: 5px;
    text-align: center;
    margin: 0 5px;
    padding: 8px 0;
    border-radius: 8px;
    background-color: rgb(83, 87, 95);
    color: white;
    border: 1px solid rgba(153, 157, 165, 0.5);
    cursor: pointer;
    transition: .5s;
}
.search-channel button:hover{
    background-color: rgb(103, 107, 115);
}
.search-channel p{
    margin: 0;
    margin-top: 10px;
}
.search-button{
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;
}
.search-button button{
    width: 20%;
    background-color: rgb(243, 130, 125) !important;
    transition: .3s;
    padding: 30px;
    cursor: pointer;
    border: 1px solid rgb(243, 130, 125);
    color: white;
    border-radius: 8px;
    padding: 8px 0;
}
.search-button button:hover{
    background-color: rgb(255, 100, 100) !important;
}
.search-genre-buttons{
  display: flex;
  align-items: center;
  width: 80%;
  margin: 0 auto;
  margin-top: 10px;
}
.search-genre-buttons p{
    width: 15%;
    margin: 0;
}
.search-genre-buttons button{
    width: 20%;
    margin: 5px;
    text-align: center;
    margin: 0 5px;
    padding: 8px 0;
    border-radius: 8px;
    background-color: rgb(83, 87, 95);
    color: white;
    border: 1px solid rgba(153, 157, 165, 0.5);
    cursor: pointer;
}
.search-genre-buttons button:hover{
    background-color: rgb(103, 107, 115);
}

ul{
  display: flex;
  justify-content: space-around;
  padding: 0;
  flex-wrap: wrap;
  transform: translateY(0px);
  transition: .5s;
  background-color: rgb(53,57,65);
  min-height: 100vh;
}
.toggled{
  transform: translateY(-530px);
}
li{
  position: relative;
  width: 23.4%;
  padding-bottom: 10px;
  margin: 10px 0.8%;
  list-style: none;
  transition: .5s;
  box-shadow: 0 0 15px 10px rgba(0, 0, 0, .2)
}
li:hover .hover-button button:nth-child(1){
    transform: skew(0deg);
}
li:hover .hover-button button:nth-child(2){
    transform: skew(0deg);
}
li img{
  width: 100%;
}
.duration-box{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  padding-top: 56.25%;
}
.duration{
  position: absolute;
  bottom: 3%;
  right: calc(3% * 0.5625);
  font-size: 0.7em;
  background-color: rgba(0,0,0,0.8);
  margin: 0;
  padding: 1px 5px;
}
.title{
  margin: 4px 15px 20px;
  font-size: 0.9em;
}
.statistics{
  position: absolute;
  font-size: 0.7em;
  opacity: 0.5;
  margin: 2px 15px;
  bottom: 7px;
}
.hover-button{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  display: flex;
  align-items: stretch;
  transition: .5s;
  color: rgba(0,0,0, 0.2);
  overflow: hidden;
  z-index: 300;
}
.hover-button::before{
  content: '';
  display: block;
  padding-top: 56.25%;
}
.hover-button button{
  background-color:rgba(53, 57, 65,0.8);
  width: 50%;
  border: none;
  transition: .5s;
  cursor: pointer;
  font-size: 1.5em;
  font-weight: bold;
  color: white;
  border: 3px solid rgba(53,57,65,0);
}
.hover-button button:hover{
  background-color: rgba(53,57,65,1);
  border: 3px solid rgba(255,255,255,0.3);
}
.hover-button button:nth-child(1) {
    transform-origin: left top;
    transform: skew(-90deg);
}
.hover-button button:nth-child(2) {
    transform-origin: right top;
    transform: skew(90deg);
}

.flip-list-move{
  transition: transform 1s;
}

@media(max-width: 1500px) {
  .container{
    left: 225px;
  }
  li{
    width: calc(100% / 3 - 1.6%);
    padding-bottom: 10px;
    margin: 10px 0.8%;
}
  }
</style>
