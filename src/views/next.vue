<template>
  <div class="next-tracks">
    <h1>{{ $t("next.nowPlaying") }}</h1>
    <TrackList
      :tracks="[currentTrack]"
      type="playlist"
      dbclickTrackFunc="none"
    />
    <h1 v-show="playNextList.length > 0">插队播放</h1>
    <TrackList
      :tracks="playNextTracks"
      type="playlist"
      :highlightPlayingTrack="false"
      dbclickTrackFunc="playTrackOnListByID"
      itemKey="id+index"
      v-show="playNextList.length > 0"
    />
    <h1>{{ $t("next.nextUp") }}</h1>
    <TrackList
      :tracks="filteredTracks"
      type="playlist"
      :highlightPlayingTrack="false"
      dbclickTrackFunc="playTrackOnListByID"
    />
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";
import { getTrackDetail } from "@/api/track";
import TrackList from "@/components/TrackList.vue";

export default {
  name: "Next",
  components: {
    TrackList,
  },
  data() {
    return {
      tracks: [],
    };
  },
  computed: {
    ...mapState(["player"]),
    currentTrack() {
      return this.player.currentTrack;
    },
    playerShuffle() {
      return this.player.shuffle;
    },
    filteredTracks() {
      let trackIDs = this.player.list.slice(
        this.player.current + 1,
        this.player.current + 100
      );
      return this.tracks.filter((t) => trackIDs.includes(t.id));
    },
    playNextList() {
      return this.player.playNextList;
    },
    playNextTracks() {
      return this.playNextList.map((tid) => {
        return this.tracks.find((t) => t.id === tid);
      });
    },
  },
  watch: {
    currentTrack() {
      this.loadTracks();
    },
    playerShuffle() {
      this.loadTracks();
    },
    playNextList() {
      this.loadTracks();
    },
  },
  methods: {
    ...mapActions(["playTrackOnListByID"]),
    loadTracks() {
      // 获取播放列表当前歌曲后100首歌
      let trackIDs = this.player.list.slice(
        this.player.current + 1,
        this.player.current + 100
      );

      // 将playNextList的歌曲加进trackIDs
      trackIDs.push(...this.playNextList);

      // 获取已经加载了的歌曲
      let loadedTrackIDs = this.tracks.map((t) => t.id);

      if (trackIDs.length > 0) {
        getTrackDetail(trackIDs.join(",")).then((data) => {
          let newTracks = data.songs.filter(
            (t) => !loadedTrackIDs.includes(t.id)
          );
          this.tracks.push(...newTracks);
        });
      }
    },
  },
  activated() {
    this.loadTracks();
  },
};
</script>

<style lang="scss" scoped>
h1 {
  margin-top: 36px;
  margin-bottom: 18px;
  cursor: default;
  color: var(--color-text);
}
</style>
