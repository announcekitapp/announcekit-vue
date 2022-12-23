<template>
    <div style="display: inline" ref="elementRef" v-bind:user="[user ? user : null]" v-bind:data="[data ? data : null]"
      v-bind:lang="[lang ? lang : null]" href="#"></div>
  </template>
  
  <script>
  import { nextTick } from 'vue'
  
  export default {
    name: "AnnounceKit",
    watch: {
      data(val) {
        if (!this.isString(val)) {
          this.loaded();
        }
      },
      user(val) {
        if (!this.isString(val)) {
          this.loaded();
        }
      },
      widgetStyle() {
        this.loaded();
      },
      embedWidget() {
        this.loaded();
      },
      floatWidget() {
        this.loaded();
      },
      lang() {
        this.loaded();
      },
      boosters() {
        this.loaded();
      },
    },
    props: {
      name: String,
      widget: {
        type: String,
        required: true,
      },
      catchClick: String,
      widgetStyle: Object,
      floatWidget: Boolean,
      embedWidget: Boolean,
      onWidgetOpen: Function,
      onWidgetClose: Function,
      onWidgetResize: Function,
      onWidgetUnread: Function,
      boosters: {
        type: Boolean,
        default: true,
      },
      user: {
        type: Object,
        default: null,
      },
      data: {
        type: Object,
        default: null,
      },
      lang: String,
    },
    data() {
      return {
        widgetHandlers: [],
      };
    },
    methods: {
      loaded() {
        const style = this.$props.widgetStyle;
  
        const styleParams = {
          badge: {
            style: style,
          },
          line: {
            style: style,
          },
          float: {
            style: style,
          },
        };
  
        if (this.$props.floatWidget) {
          delete styleParams.badge;
          delete styleParams.line;
        }
  
        this.widgetName = Math.random().toString(36).substring(10);
  
        const _this = this;
  
        const options = Object.assign({}, styleParams, {
          widget: this.$props.widget,
          name: this.widgetName,
          version: 2,
          framework: "vue",
          framework_version: "3.0.0",
          selector: _this.elementRef,
          embed: this.$props.embedWidget,
          onInit: function (_widget) {
            if (_widget.conf.name !== _this.widgetName) {
              return _widget.destroy();
            }
  
            const ann = window["announcekit"];
  
            _this.widgetInstance = _widget;
  
            _this.widgetHandlers.forEach((h) => h(_widget));
            _this.widgetHandlers = [];
  
            if (_this.$props.catchClick) {
              const elem = document.querySelector(_this.$props.catchClick);
  
              if (elem)
                elem.addEventListener("click", function () {
                  _widget.open();
                });
            }
  
            ann.on("widget-open", function (ref) {
              if (ref.widget === _widget && _this.$props.onWidgetOpen) {
                _this.$props.onWidgetOpen({ widget: ref.widget });
              }
            });
  
            ann.on("widget-close", function (ref) {
              if (ref.widget === _widget && _this.$props.onWidgetClose) {
                _this.$props.onWidgetClose({ widget: ref.widget });
              }
            });
  
            ann.on("widget-resize", function (ref) {
              if (ref.widget === _widget && _this.$props.onWidgetResize) {
                _this.$props.onWidgetResize({
                  widget: ref.widget,
                  size: ref.size,
                });
              }
            });
  
            if (_this.$props.onWidgetUnread) {
              _this.$props.onWidgetUnread({
                widget: _widget,
                unread: _widget.state.ui.unreadCount,
              });
            }
          },
          data: JSON.parse(JSON.stringify(_this.$props.data)),
          user: JSON.parse(JSON.stringify(_this.$props.user)),
          lang: _this.$props.lang,
          boosters: _this.$props.boosters,
        });
  
  
        window["announcekit"].push(options);
      },
      open() {
        this.withWidget((widget) => widget.open());
      },
      close() {
        this.withWidget((widget) => widget.close());
      },
      unread() {
        return this.withWidget((widget) => widget.state.ui.unreadCount);
      },
      instance() {
        return this.withWidget((widget) => widget);
      },
      withWidget(fn) {
        return new Promise((res) => {
          if (this.widgetInstance) {
            return res(fn(this.widgetInstance));
          } else {
            this.widgetHandlers.push((widget) => {
              res(fn(widget));
            });
          }
        });
      },
      isString(obj) {
        return obj !== undefined && obj !== null && obj.constructor === String;
      },
    },
    created() {
      this.widgetInstance = null;
    },
    async mounted() {
      await nextTick();
      if (!window["announcekit"]) {
        window["announcekit"] = window["announcekit"] || {
          queue: [],
          push: function (x) {
            window["announcekit"].queue.push(x);
          },
          on: function (n, x) {
            window["announcekit"].queue.push([n, x]);
          },
        };
  
        var scripttag = document.createElement("script");
        scripttag["async"] = true;
        scripttag["src"] = `https://cdn.announcekit.app/widget-v2.js`;
  
        var scr = document.getElementsByTagName("script")[0];
        scr.parentNode.insertBefore(scripttag, scr);
      }
      this.elementRef = this.$refs.elementRef;
      this.loaded();
    },
    async updated() {
      await nextTick();
      if (this.widgetInstance) {
        this.widgetInstance.destroy();
        this.loaded();
      }
    },
  };
  </script>