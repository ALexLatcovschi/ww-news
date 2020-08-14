<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="section">
        <!-- wwManager:start -->
        <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <div class="news-section wwbuilder">
            <div class="header container ww-column">
                <div class="topic">
                    <wwObject class="text" v-bind:ww-object="section.data.topic"></wwObject>
                </div>
                <div class="link">
                    <a href="https://medium.com/42cap" target="_blank">
                        <wwObject class="text" v-bind:ww-object="section.data.mediumNews"></wwObject>
                        <i class="fas fa-arrow-right"></i>
                    </a>
                </div>
            </div>
            <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.contentList" class="list"
                            @ww-add="add(section.data.contentList, $event)"
                            @ww-remove="remove(section.data.contentList, $event)">
                <div class="slick-news">
                    <div class="overflow-block">
                        <div class="custom-slide" v-for="(item, index) in newsData" :key="index">
                            <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.contentList"
                                            class="list"
                                            @ww-add="add(section.data.contentList, $event)"
                                            @ww-remove="remove(section.data.contentList, $event)">
                                <div class="round-item"></div>
                                <div class="info-block">
                                    <div class="title">
                                        <a :href="item.link" target="_blank">
                                            <wwObject class="text" v-bind:ww-object="item.title"></wwObject>
                                        </a>
                                    </div>
                                    <div class="author">
                                        <wwObject class="text" v-bind:ww-object="item.author"></wwObject>
                                    </div>
                                </div>
                            </wwLayoutColumn>
                        </div>
                    </div>
                </div>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
    import axios from 'axios';
    import VueSlickCarousel from 'vue-slick-carousel';
    import 'vue-slick-carousel/dist/vue-slick-carousel.css';
    // optional style for arrows & dots
    import 'vue-slick-carousel/dist/vue-slick-carousel-theme.css';

    export default {
        name: '__COMPONENT_NAME__',
        props: {
            // The section controller object is passed. You can access it anytime
            sectionCtrl: Object,
            newsData: []
        },
        data() {
            return {
                settings: {
                    infinite: false,
                    speed: 500,
                    slidesToShow: 3,
                    slidesToScroll: 1
                }
            };
        },
        computed: {
            // The section object contains all the info and data about the section
            // Use it as you like !
            section() {
                return this.sectionCtrl.get();
            }
        },
        async created() {
            // Initialize the data once the section is created in the DOM
            this.init();
            await this.getMediumData();
        },
        beforeUpdate() {
            if (this.$refs.VueSlickCarousel) {
                this.$refs.VueSlickCarousel.destroy();
            }
        },
        updated() {
            this.$nextTick(function() {
                if (this.$refs.VueSlickCarousel) {
                    this.$refs.VueSlickCarousel.create(this.settings);
                }
            });
        },
        methods: {
            init() {
                // Initialize section data
                let needUpdate = false;

                // We will only save the data in this.section.data
                // So you need to put in there all the data of you WeWeb objects
                this.section.data = this.section.data || {};

                // Initialize WeWeb objects that are in the html template up there
                if (!this.section.data.background) {
                    this.section.data.background = wwLib.wwObject.getDefault({ type: 'ww-color' });
                    needUpdate = true;
                }

                if (!this.section.data.title) {
                    this.section.data.title = wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                fr: 'Hello World FR !',
                                en: 'Hello World !'
                            }
                        }
                    });
                    needUpdate = true;
                }
                if (!this.section.data.topic) {
                    this.section.data.topic = wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: 'News'
                            }
                        }
                    });
                    needUpdate = true;
                }
                if (!this.section.data.mediumNews) {
                    this.section.data.mediumNews = wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: 'More on Medium'
                            }
                        }
                    });
                    needUpdate = true;
                }

                if (!this.section.data.image) {
                    this.section.data.image = wwLib.wwObject.getDefault({ type: 'ww-image' });
                    needUpdate = true;
                }

                if (!this.section.data.contentList) {
                    this.section.data.contentList = [];
                    needUpdate = true;
                }

                if (needUpdate) {
                    this.sectionCtrl.update(this.section);
                }
            },
            // --------- EDITOR FUNCTIONS ---------
            // All the codes between /* wwManager:start */ and /* wwManager:end */ are only for editor purposes
            // So It won't in the published website!
            /* wwManager:start */
            add(list, options) {
                try {
                    list.splice(options.index, 0, options.wwObject);
                    this.sectionCtrl.update(this.section);
                } catch (error) {
                    wwLib.wwLog.error('ERROR : ', error);
                }
            },
            remove(list, options) {
                try {
                    list.splice(options.index, 1);
                    this.sectionCtrl.update(this.section);
                } catch (error) {
                    wwLib.wwLog.error('ERROR : ', error);
                }
            },
            /* wwManager:end */

            async getMediumData() {
                const response = await axios.get(`https://api.rss2json.com/v1/api.json?rss_url=https://medium.com/feed/42cap`);
                this.newsData = response.data.items.slice(1).slice(-4);
                console.log('news data: ', this.newsData);
                this.newsData.map(el => {
                    el.title = wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: el.title
                            }
                        }
                    });
                    el.author = wwLib.wwObject.getDefault({
                        type: 'ww-text',
                        data: {
                            text: {
                                en: el.author
                            }
                        }
                    });
                });
            }
        },
        components: { VueSlickCarousel }
    };
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss" scoped>
    .section {
        pointer-events: all;
    }

    .news-section {
        padding: 24px 0 38px;
        color: #FFFFFF;
        background: #343C38;

        .header {
            display: flex;
            margin: 0 auto 27px;

            .topic {
                font-style: normal;
                font-weight: bold;
                font-size: 48px;
                align-items: flex-end;
                display: flex;
                line-height: 38px;
            }

            .link {
                font-style: normal;
                font-weight: normal;
                font-size: 18px;
                line-height: 21px;
                display: flex;
                align-items: flex-end;
                margin-left: 10px;

                a {
                    display: flex;
                    color: white;
                    outline: none;
                    border: none;
                    text-decoration: none;

                    i {
                        margin-left: 10px;
                        border: none;
                        text-decoration: none;

                        &:before {
                            outline: none;
                            border: none;
                            text-decoration: none;
                        }
                    }
                }
            }
        }

        .slick-news {
            .custom-slide {
                cursor: pointer;
                display: flex !important;
                align-items: center;

                .round-item {
                    width: 18px;
                    height: 18px;
                    background: #FFFFFF;
                    margin: 0 32px;
                    border-radius: 50px;
                }

                .info-block {
                    .title {
                        width: max-content;

                        a {
                            color: white;
                        }
                    }

                    .author {
                        font-style: normal;
                        font-weight: normal;
                        font-size: 12px;
                        line-height: 14px;
                        color: #67836F;
                        margin-top: 4px;
                    }
                }
            }
        }

        .overflow-block {
            display: flex;
            overflow-x: scroll;
            -ms-overflow-style: none;
            scrollbar-width: none;

            .list {
                display: flex;
            }

            &::-webkit-scrollbar {
                display: none;
            }

            &::-webkit-scrollbar {
                height: 80px;
            }

            &::-webkit-scrollbar-track {
                background: red;
                border-radius: 10px;
            }

            &::-webkit-scrollbar-thumb {
                background: blue;
                border-radius: 10px;
            }
        }
    }
</style>
