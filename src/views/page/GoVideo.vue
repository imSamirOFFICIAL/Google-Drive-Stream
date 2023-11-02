<template>
  <div class="content g2-content">
    <div v-if="options && options.api" class="video-content">
      <iframe
        width="100%"
        height="100%"
        :src="apiVideoUrl"
        frameborder="0"
        border="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        allowtransparency="true"
        allowfullscreen="true"
      ></iframe>
    </div>
    <div v-else>
      <vue-plyr ref="plyr" :options="options">
        <video controls crossorigin playsinline>
          <source :src="videoUrl" type="video/mp4" />
          <track
            kind="captions"
            label="Default"
            srclang="default"
            :src="subtitle"
            default
          />
        </video>
      </vue-plyr>
    </div>
  </div>
</template>

<script>
import { decode64 } from "@utils/AcrouUtil";
import VuePlyr from "vue-plyr";
export default {
  comments: {
    VuePlyr,
  },
  data: function() {
    return {
      apiVideoUrl: "",
      videoUrl: "",
      subtitle: "",
    };
  },
  mounted() {
    this.render();
  },
  methods: {
    render() {
      let path = encodeURI(this.url);
      let index = path.lastIndexOf(".");
      this.suffix = path.substring(index + 1, path.length);
      this.loadSub(path, index);
      this.videoUrl = window.location.origin + path;
      this.apiVideoUrl = this.options.api + this.videoUrl;
      if (!this.options.api) {
        let options = {
          src: this.videoUrl,
          autoplay: this.options.autoplay,
          media: this.player.media,
        };
        if (this.suffix === "m3u8") {
          this.loadHls(options);
        } else if (this.suffix === "flv") {
          this.loadFlv(options);
        }
      }
    },
    loadSub(path, index) {
      this.subtitle = path.substring(0, index) + ".vtt";
    },
    loadHls(options) {
      import("@/plugin/vplayer/hls").then((res) => {
        var Hls = res.default;
        Hls({
          ...options,
          callback: (hls) => {
            // Handle changing captions
            this.player.on("languagechange", () => {
              // Caption support is still flaky. See: https://github.com/sampotts/plyr/issues/994
              setTimeout(
                () => (hls.subtitleTrack = this.player.currentTrack),
                50
              );
            });
          },
        });
      });
    },
    loadFlv(options) {
      import("@/plugin/vplayer/flv").then((res) => {
        var Flv = res.default;
        Flv(options);
      });
    },
    copy() {
      this.$copyText(this.videoUrl);
    },
  },
  computed: {
    options() {
      var options = window.themeOptions.video;
      return {
        autoplay: false,
        invertTime: false,
        settings: ["quality", "speed", "loop"],
        ratio: "16:9",
        controls: [
          "play-large",
          "restart",
          "play",
          "progress",
          "current-time",
          "duration",
          "mute",
          "volume",
          "captions",
          "settings",
          "pip",
          "airplay",
          "fullscreen",
        ],
        ...options,
        captions: { active: true, language: "default", ...options.captions },
      };
    },
    player() {
      return this.$refs.plyr.player;
    },
  },
};
</script>
