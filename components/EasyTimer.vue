<template>
    <div class="flex flex-col items-center justify-center h-screen">
        <div class="p-20 shadow-xl border border-gray">
            <div v-if="!isRunning" class="text-center">
                <div class="flex items-center mb-2">
                    <input type="text" id="timeInput" v-model="timeInput" placeholder="Enter Time here -- Example 1h30m"
                        class="w-full p-5 border rounded-lg" />
                </div>
                <button @click="startTimer" :disabled="isRunning"
                    class="px-8 py-3 bg-blue-500 text-white rounded-2xl disabled:bg-gray-400 disabled:cursor-not-allowed w-full">
                    Start
                </button>
            </div>
            <div v-else class="text-center">
                <p class="text-4xl font-bold mb-4">Time remaining:</p>
                <p class="text-6xl font-bold">{{ timeDisplay }}</p>
                <button @click="stopTimer"
                    class="px-4 py-2 mt-4 bg-red-500 text-white rounded-lg disabled:bg-gray-400 disabled:cursor-not-allowed">
                    Stop
                </button>
            </div>
        </div>
        <audio ref="alertSound" src="/Daybreak.mp3"></audio>
    </div>
</template>
  
<script>
export default {
    data() {
        return {
            timeInput: "",
            time: 0,
            isRunning: false,
        };
    },
    computed: {
        timeDisplay() {
            const minutes = Math.floor(this.time / 60);
            const seconds = this.time % 60;
            return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        },
    },
    mounted() {
        this.autoStartTimerFromQueryParams();
    },
    beforeUnmount() {
        this.pauseAlertSound();
    },
    methods: {
        parseTimeInput(input) {
            const timeRegex = /(\d+h)?(\d+m)?(\d+s)?/;
            const matches = input.match(timeRegex);
            let totalSeconds = 0;

            if (matches) {
                const hours = parseInt(matches[1]) || 0;
                const minutes = parseInt(matches[2]) || 0;
                const seconds = parseInt(matches[3]) || 0;

                totalSeconds = hours * 3600 + minutes * 60 + seconds;
            }

            return totalSeconds;
        },
        startTimer() {
            this.time = this.parseTimeInput(this.timeInput);
            if (this.time > 0 && !this.isRunning) {
                this.isRunning = true;
                this.countdown();
            }
        },
        countdown() {
            if (this.time > 0) {
                setTimeout(() => {
                    this.time--;
                    this.countdown();
                }, 1000);
            } else {
                this.playAlertSound();
                this.isRunning = false;
            }
        },
        stopTimer() {
            this.isRunning = false;
            this.timeInput = "";
            this.time = 0;
        },
        autoStartTimerFromQueryParams() {
            const urlParams = new URLSearchParams(window.location.search);
            const timeParam = urlParams.get("t");

            if (timeParam) {
                this.timeInput = timeParam;
                this.startTimer();
            }
        },
        playAlertSound() {
            const audioElement = this.$refs.alertSound;
            audioElement.play();
        },
        pauseAlertSound() {
            const audioElement = this.$refs.alertSound;
            audioElement.pause();
            audioElement.currentTime = 0;
        }
    },
};
</script>
  
  