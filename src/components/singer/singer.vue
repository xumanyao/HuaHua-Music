<template>
  <div class="singer">
    <list-view :data="singers" @selectItem="selectSinger"></list-view>
    <router-view></router-view>
  </div>
</template>

<script>
  import {getSingerList} from 'api/singer';
  import {ERR_OK} from 'api/config';
  import Singer from 'common/js/singer';
  import ListView from 'base/listView/listview';

  const HOT_NAME = '热门';
  const HOT_SINGER_LEN = 10;
  export default {
    name: 'singer',
    components: {
      ListView
    },
    data() {
      return {
        singers: []
      };
    },
    created() {
      this._getSingerList();
    },
    methods: {
      selectSinger(singer) {
        this.$router.push({
          path: `/singer/${singer.id}`
        });
      },
      _getSingerList() {
        getSingerList().then((res) => {
          if (res.code === ERR_OK) {
            // this.singers = res.data.list;
            // console.log(this._normalizeSinger(this.singers));
            this.singers = this._normalizeSinger(res.data.list);
          }
        });
      },
      _normalizeSinger(list) {
        let map = {
          hot: {
            title: HOT_NAME,
            items: []
          }
        };
        list.forEach((item, index) => {
          if (index < HOT_SINGER_LEN) {
            map.hot.items.push(new Singer({
              id: item.Fsinger_mid,
              name: item.Fsinger_name
            }));
          }
          const key = item.Findex;
          if (!map[key]) {
            map[key] = {
              title: key,
              items: []
            };
          }
          map[key].items.push(new Singer({
            id: item.Fsinger_mid,
            name: item.Fsinger_name
          }));
        });
        let hot = [];
        let ret = [];
        for (let key in map) {
          if (map[key].title.match(/[a-zA-Z]/)) {
            ret.push(map[key]);
          } else if (map[key].title === HOT_NAME) {
            hot.push(map[key]);
          }
        }
        ret.sort((a, b) => {
          return a.title.charCodeAt(0) - b.title.charCodeAt(0);
        });
        return hot.concat(ret);
      }

    }
  };
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .singer
    position: fixed
    top: 80px
    bottom: 0
    width: 100%
</style>
