<template>
  <div class="daily">
    <div class="daily-menu">
      <div class="daily-menu-item"
           :class="{on:type === 'recommend'}"
           @click="handleToRecommend">每日推荐
      </div>
      <div class="daily-menu-item"
           :class="{on:type === 'daily'}"
           @click="showThemes = !showThemes">主题日报
      </div>
      <ul v-show="showThemes">
        <li v-for="item in themes">
          <a :class="{on:item.id === themeId && type === 'daily'}"
             @click="handleToTheme(item.id)">{{item.name}}</a>
        </li>
      </ul>
    </div>
    <div class="daily-list" ref="list">
      <template v-if="type === 'recommend'">
        <div v-for="list in recommendList">
          <div class="daily-date">{{ formatDay(list.date) }}</div>
          <Item
              v-for="item in list.stories"
              :data="item"
              :key="item.id"
              @click.native="handleClick(item.id)"></Item>
        </div>
      </template>
      <template v-if="type === 'daily'">
        <Item
            v-for="item in list"
            :data="item"
            :key="item.id"
            @click.native="handleClick(item.id)"></Item>
      </template>
    </div>
    <daily-article :id="articleId"></daily-article>
  </div>
</template>
<script>
  import $ from './libs/util';
  import Item from './components/item.vue';
  import dailyArticle from './components/daily-article.vue';

  export default {
    components: {Item, dailyArticle},
    data() {
      return {
        type: 'recommend',
        recommendList: [],
        themes: [],
        list: [],
        dailyTime: $.getTodayTime(),
        isLoading: false,
        showThemes: false,
        themeId: 0,
        articleId: 0
      }
    },
    methods: {
      getThemes() {
        //axios 发起 get 请求
        $.ajax.get('themes').then(res => {
          this.themes = res.others;
        })
      },
      handleToTheme() {
        //改变菜单分类
        this.type = 'daily';
        //设置当前电子子类的主题日报id
        this.themeId = id;
        //清空中间栏的数据
        this.list = [];
        $.ajax.get('theme/' + id).then(res => {
          //过滤掉类型为1的文章,该类型下的文章为空
          this.list = res.stories.filter(item => item.type !== 1);
        })
      },
      handleToRecommend() {
        this.type = 'recommend';
        this.recommendList = [];
        this.dailyTime = $.getTodayTime();
        this.getRecommendList();
      },
      getRecommendList() {
        this.isLoading = true;
        const prevDay = $.prevDay(this.dailyTime + 86400000);
        $.ajax.get('news/before/' + prevDay).then(res => {
          this.recommendList.push(res);
          this.isLoading = false;
        })
      },
      formatDay(date) {
        let month = date.substr(4, 2);
        let day = date.substr(6, 2);
        if (month.substr(0, 1) === '0') month = month.substr(1, 1);
        if (day.substr(0, 1) === '0') date = day.substr(1, 1);
        return `${month}月${day}日`;
      },
      getRecommendList() {
        //加载时设置为true,加载完成后置为false
        this.isLoading = true;
        const prevDay = $.prevDay(this.dailyTime + 86400000);
        $.ajax.get('news/before/' + prevDay).then(res => {
          this.recommendList.push(res);
          this.isLoading = false;
        })
      },
      handleClick(id) {
        this.articleId = id;
      },
      handleToTheme(id) {
        this.type = 'daily';
        this.themeId = id;
        this.list = [];
        $.ajax.get('theme/' + id).then(res => {
          this.list = res.stories
            .filter(item => item.type !== 1);
        })
      }
    },
    mounted() {
      this.getRecommendList();
      this.getThemes();
      const $list = this.$refs.list;
      $list.addEventListener('scroll', () => {
        //在'主题日报'或正在加载推荐列表时停止操作
        if (this.type === 'daily' || this.isLoading) return;
        if
        (
          $list.scrollTop
          + document.body.clientHeight
          >= $list.scrollHeight
        ) {
          this.dailyTime -= 86400000;
          this.getRecommendList();
        }
      });
    }
  }
</script>