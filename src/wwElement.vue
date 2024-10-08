<template>
    <div class="ww-video" :class="{ editing: isEditing }">
        <div class="ww-video-container">
            <video
                v-if="isWeWeb"
                ref="videoPlayer"
                class="ww-video-element"
                playsinline
                webkit-playsinline
                v-bind="videoAttributes"
            >
                Sorry, your browser doesn't support embedded videos.
            </video>
            <iframe
                v-else
                class="ww-video-element"
                :src="src"
                frameborder="0"
                webkitallowfullscreen
                mozallowfullscreen
                allowfullscreen
                scrolling="no"
                loading="lazy"
            ></iframe>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        content: { type: Object, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
    },
    emits: ['update:content:effect'],
    data() {
        return {
            isVideoPlayed: false,
            isEventListener: false,
        };
    },
    updated() {
        console.log('UPDATE');
    },
    computed: {
        videoElement() {
            return this.isWeWeb ? this.$refs.video : null;
        },
        isWeWeb() {
            return this.content.provider === 'weweb';
        },
        src() {
            if (!this.content.url) return;

            let src = this.content.url;
            const provider = this.getInfoFromUrl(src).provider;

            const id = this.getInfoFromUrl(src).id;
            if (provider === 'other' || provider === 'local') return;
            switch (provider) {
                case 'youtube':
                    src = `//www.youtube.com/embed/${id}?rel=0`;
                    break;
                case 'twitch':
                    src = `//player.twitch.tv/?video=${id}&parent=${window.location.hostname}`;
                    break;
                case 'dailymotion':
                    src = `//www.dailymotion.com/embed/video/${id}?`;
                    break;
                case 'vimeo':
                    src = `//player.vimeo.com/video/${id}?`;
                    break;
                default:
                    break;
            }
            if (provider === 'youtube') {
                if (this.content.loop) src += `&loop=1&playlist=${id}`;
                if (this.content.autoplay) src += '&autoplay=1';
                if (this.content.muted) src += '&mute=1';
                if (!this.content.controls) src += '&controls=0';
            } else if (provider === 'twitch') {
                src += `&autoplay=${this.content.autoplay}`;
                src += `&mute=${this.content.muted}`;
            } else {
                if (this.content.muted) src += '&muted=1';
                if (!this.content.controls) src += '&controls=0';
                if (this.content.autoplay) src += '&autoplay=1';
                if (this.content.loop) src += '&loop=1';
            }

            return src;
        },
        isPreviewImageWeWeb() {
            return this.content.previewImage && this.content.previewImage.startsWith('designs/');
        },
        previewImageSrc() {
            return this.isPreviewImageWeWeb
                ? `${wwLib.wwUtils.getCdnPrefix()}${this.content.previewImage}`
                : this.content.previewImage;
        },
        videoAttributes() {
            const attributes = {
                src: this.content.file,
                poster: this.previewImageSrc,
                muted: true,
            };

            if (this.content.autoplay) attributes.autoplay = true;
            if (this.content.muted) attributes.muted = true;
            if (this.content.controls) attributes.controls = true;
            if (this.content.loop) attributes.loop = true;
            if (this.content.preload) attributes.preload = true;

            return attributes;
        },

        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
    },
    watch: {
        'content.provider'() {
            this.$emit('update:content:effect', {
                url: '',
                file: '',
                previewImage: '',
                preload: '',
            });
        },
        'content.autoplay'(newAuto, oldAuto) {
            if (this.content.autoplay) {
                this.$emit('update:content:effect', {
                    muted: true,
                });
            }
            if (newAuto !== oldAuto && newAuto) {
                this.updateweWeWebVideo();
            }
        },
        'content.loop'(newLoop, oldLoop) {
            if (newLoop !== oldLoop && newLoop) {
                this.updateweWeWebVideo();
            }
        },
    },
    unmounted() {
        if (this.isEventListener) {
            const videoEl = document.querySelector('.ww-video-element');
            videoEl.removeEventListener('click', this.handleVideoClick);
            this.isEventListener = false;
        }
    },
    methods: {
        getInfoFromUrl(url) {
            if (!this.content.url) return {};
            if (url.indexOf('youtube.com') !== -1) {
                return {
                    id: url.split('v=')[1].split('?')[0],
                    provider: 'youtube',
                };
            } else if (url.indexOf('youtu.be') !== -1) {
                return {
                    id: url.split('be/')[1].split('?')[0],
                    provider: 'youtube',
                };
            } else if (url.indexOf('vimeo.com') !== -1) {
                return {
                    id: url.split('m/')[1].split('?')[0],
                    provider: 'vimeo',
                };
            } else if (url.indexOf('dailymotion.com') !== -1) {
                return {
                    id: url.split('video/')[1].split('?')[0],
                    provider: 'dailymotion',
                };
            } else if (url.indexOf('twitch.tv') !== -1) {
                return {
                    id: url.split('tv/videos/')[1].split('?')[0],
                    provider: 'twitch',
                };
            } else {
                return {
                    id: url,
                    provider: 'other',
                };
            }
        },
        updateweWeWebVideo() {
            if (this.content.provider !== 'weweb' || !this.$refs.videoPlayer) return;

            const video = this.$refs.videoPlayer;
            video.pause();
            video.currentTime = 0;
            video.play();
        },
    },
};
</script>

<style lang="scss" scoped>
.ww-video {
    position: relative;
    /* wwEditor:start */
    &.editing::before {
        content: '';
        inset: 0;
        position: absolute;
        z-index: 1;
    }
    /* wwEditor:end */
    .ww-video-container {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;

        .ww-video-element {
            position: relative;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }
    }
}
</style>
