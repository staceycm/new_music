<template>
  <div class="main">
    <van-list v-model="loading" :finished="finished" finished-text="没有更多了" @load="onLoad">
      <van-sticky :offset-top="44">
        <div v-if="loading1 == true" class="header"  @click="startPlayall">
          <div class="icon">
            <van-icon size="20px" name="play-circle-o" />
          </div>
          <div class="content1">
            <div class="name1">
              <span class="title">播放全部</span>
              <span class="count">(共{{songsCount}}首)</span>
            </div>
          </div>
          <div class="sub_btn">
            <van-button type="danger" size="small" round>+收藏({{2}})</van-button>
          </div>
        </div>
      </van-sticky>
      <div @click="fullScreen(item,index)" class="lists" v-for="(item, index) in this.songs" :key="index">
        <div class="index">
          <div>{{index + 1}}</div>
        </div>
        <div class="content van-ellipsis">
          <div class="name van-ellipsis">
            <span>{{item.name}}</span>
            <span v-if="item.name == ''">未知</span>
            <span class="alias" v-if="item.alia != ''">({{item.alia[0]}})</span>
          </div>
          <div class="desc van-ellipsis">
            <span>{{item.ar[0].name}}</span>
            <span>-</span>
            <span class="zhuanji">{{item.al.name}}</span>
          </div>
        </div>
        <div class="score">
          <van-icon v-if="item.mv !== 0" name="video-o" />
        </div>
        <div class="score">
          <van-icon class="icon2" name="ellipsis" />
        </div>
      </div>
    </van-list>
  </div>
</template>

<script>
export default {
  data() {
    return {
      songs: [],
      page: 0,
      number: 15,
      loading: false,
      finished: false,
      songsCount: "",
      loading1: false,
      allSongList: []
    };
  },
  methods: {
    onLoad() {
      setTimeout(async () => {
        await this.$http
          .get("/album?id=" + this.$store.state.musiclistid)
          .then(res => {
            this.allSongList = res.data.songs;
            this.songsCount = res.data.songs.length;
            var date = res.data.songs.slice(this.page, this.number);
            date.forEach(element => {
              this.songs.push(element);
            });
          });
        this.loading = false;
        this.loading1 = true;
        //数据全部加载完毕
        if (this.songs.length >= this.songsCount) {
          //让正在加载中的提示隐藏
          this.finished = true;
        }
        this.page = this.page + 15;
        this.number = this.number + 15;
      }, 1000);
    },
    startPlayall() {
      console.log(this.allSongList[0].id);
      this.$store.dispatch("startPlayAll", this.allSongList);
      this.$store.commit("audio_ing_song",  [this.allSongList[0]]);
      this.$store.commit("startMusic", this.allSongList[0].id);
      this.$store.commit("isFull", true);
    },
    fullScreen(item,index) {
      this.$store.commit('returnIndex', index);
      this.$http("/song/detail?ids=" + item.id).then(res => {
        this.$store.dispatch("startPlayAll", this.allSongList);
        this.$store.commit("audio_ing_song", res.data.songs);
        this.$store.commit("startMusic", item.id);
        this.$store.commit("isFull", true);
      });
    }
  }
};
</script>

<style scoped>
.main {
  margin-top: -15px;
  position: relative;
  z-index: 1;
}
.header {
  height: 44px;
  width: 100%;
  display: flex;
  align-items: center;
  background: white;
  border-radius: 15px 15px 0px 0px;
}
.header .icon {
  height: 44px;
  width: 44px;
  position: relative;
}
.icon .van-icon {
  position: absolute;
  transform: translateY(-50%) translateX(-50%);
  top: 50%;
  left: 50%;
}
.count {
  font-size: 12px;
  color: #a2a2a2;
}
.sub_btn {
  width: 100%;
  height: 44px;
  position: absolute;
}
.sub_btn .van-button {
  position: absolute;
  transform: translateY(-50%);
  top: 50%;
  right: 6px;
}
.title {
  font-size: 15px;
}
.lists {
  height: 50px;
  display: flex;
  align-items: center;
}
.lists:active {
  background: #e0e0e0;
}
.index,
.score {
  width: 44px;
  height: 50px;
  position: relative;
  color: #a3a3a3;
}
.score {
  width: 30px;
}
.content {
  flex: 1;
  font-size: 13px;
  width: 100%;
}
.name {
  position: relative;
}
.alias {
  color: #a2a2a2;
}
.index div {
  position: absolute;
  transform: translateY(-50%) translateX(-50%);
  top: 50%;
  left: 50%;
  font-size: 13px;
}
.score .van-icon {
  position: absolute;
  transform: translateY(-50%) translateX(-50%);
  top: 50%;
  left: 50%;
  font-size: 20px;
}
.desc {
  font-size: 10px;
  color: #a2a2a2;
}
</style>