<template>
    <div class="index">
        <div class="typeStatus">
            Chars: {{ currentChars }} Words: {{ currentWords }} CPM: {{ currentCPM.toFixed(0) }} WPM: {{
            currentWPM.toFixed(0) }} Time left: {{ 60 - timeTaken }}
            <a href="/">
                <div class="duringRestart">Restart</div>
            </a>
        </div>
        <div class="paragraphInput" id="paragraphInput">
            <span
                v-for="word in paragraph"
                :class="{
                    paraWord: true,
                    error: word.status === 'error',
                    correct: word.status === 'check',
                    current: word.status === 'current'
                }"
                :id="word.status === 'current' ? 'currentWord' : word.status === 'next' ? 'next' : 'word'"
            >
                {{ word.word }}
            </span>
        </div>
        <input
            class="typingInput"
            v-model="currentInput"
        />
        <div>
            <br>
            <h3>How do I type faster?</h3>
            <p>
                Make sure you learn touch typing. It's a typing method that utilizes all 10 of your fingers in order
                to type the fastest speed possible. You could also try switching to Dvorak key layout, but I found that
                that didn't make too much of a difference. Otherwise, just take a deep breath, focus on typing, and go
                your best. There's no "endurance" here, your fingers won't wear out in a minute. Just push yourself to
                the limit, as you can always take a break afterwards.
            </p>
            <h3>Can I read about this site?</h3>
            <p>
                Sure! Just click on the hamburger icon on the left and navigate to the "About" tab. You'll find more
                information about typing and this site there.
            </p>
        </div>
        <modal name="complete" class="completeModal" :clickToClose="false">
            <div class="modalContent">
                <h2>Typing complete!</h2>
                <br>
                <p>
                    In 1 minute, you typed {{ currentChars }} characters and {{ currentWords }} words. {{ currentWords
                    >= 100 ? "Wow! That's a lot!" : "Keep practicing and you'll get even better!"}}
                </p>
                <p>
                    You have taken this test {{ prevTimes.length }} times. This test was place #{{ place }}.
                </p>
                <a href="/">
                    <div class="modalRefreshButton">
                        Try again
                    </div>
                </a>
            </div>
        </modal>
    </div>
</template>

<script lang="ts">
    import {Component, Vue} from "vue-property-decorator";

    const randomWords = require('random-words');

    import VModal from 'vue-js-modal';

    Vue.use(VModal, {dynamic: true, dymanicDefaults: {clickToClose: false}});

    import {sourcewords} from "~/assets/words";

    interface WordData {
        word: string
        status: "check" | "error" | "current" | "next" | "na"
    }

    interface SpeedRecord {
        wpm: number,
        cpm: number,
        time: number
    }

    @Component({})
    export default class index extends Vue {
        currentInput = "";
        words: Array<string> = this.generateWords(sourcewords, 500);
        paragraph: Array<WordData> = this.words.map(l => ({
            word: l,
            status: "na"
        }));
        currentWord = -1;
        timeTaken = 0;
        currentChars = 0;
        currentCPM = 0;
        currentWords = 0;
        currentWPM = 0;
        timer: any;
        prevTimes: Array<SpeedRecord> = [];
        place: number = 0;

        generateWords(arr: Array<string>, amount: number) {
            let res = [];
            for (let i = 0; i < amount; i++) {
                res.push(arr[Math.floor(Math.random() * arr.length)]);
            }
            return res;
        }

        scrollToWord() {
            let nextEle = document.getElementById("next");
            if (nextEle) {
                if (nextEle.offsetTop > 67) {
                    const ele = document.getElementById("paragraphInput");
                    if (ele) {
                        ele.scrollTop = nextEle.offsetTop - 126;
                    }
                }
            }
        }

        startTimer() {
            this.timer = setInterval(() => {
                if (this.timeTaken === 60) {
                    this.prevTimes.push({
                        cpm: this.currentCPM,
                        wpm: this.currentWPM,
                        time: Date.now()
                    });
                    console.log(this.prevTimes.sort((a, b) => b.wpm - a.wpm));
                    this.place = this.prevTimes.sort((a, b) => b.cpm - a.cpm).findIndex(l => l.cpm === this.currentCPM) + 1;
                    localStorage.setItem("times", JSON.stringify(this.prevTimes));
                    clearInterval(this.timer);
                    this.$modal.show("complete");
                } else {
                    this.timeTaken++;
                }
            }, 1000)
        }

        updateCWPM() {
            this.currentCPM = this.timeTaken !== 0 ? this.currentChars / this.timeTaken * 60 : 0;
            this.currentWPM = this.timeTaken !== 0 ? this.currentWords / this.timeTaken * 60 : 0;
        }

        mounted() {
            const storage = localStorage.getItem("times");
            if (storage !== null) {
                this.prevTimes = JSON.parse(storage);
            }
            window.addEventListener("keypress", e => {
                if (this.currentInput.trim() !== "") {
                    if (this.currentWord === -1) {
                        this.currentWord = 0;
                        this.paragraph[this.currentWord].status = "current";
                        this.startTimer();
                    } else {
                        if (e.key === " ") {
                            if (this.currentInput.trim() === this.paragraph[this.currentWord].word) {
                                this.paragraph[this.currentWord].status = "check";
                                this.currentWords++;
                            } else {
                                this.paragraph[this.currentWord].status = "error";
                            }
                            if (this.currentWord + 1 < this.paragraph.length) {
                                this.currentChars += this.paragraph[this.currentWord].word.length;
                                this.currentWord++;
                                this.paragraph[this.currentWord].status = "current";
                                if (this.currentWord + 1 < this.paragraph.length) {
                                    this.paragraph[this.currentWord + 1].status = "next";
                                }
                            }
                            this.currentInput = "";
                            this.scrollToWord();
                            this.updateCWPM();
                        }
                    }
                } else {
                    if (e.key === " ") {
                        this.currentInput = "";
                    }
                }
            })
        }
    }
</script>

<style scoped lang="scss">
    .typeStatus {
        margin: 10px;
        width: 100%;
        border: 2px solid;
        padding: 10px;
        text-align: center;
        font-size: 18px;
    }

    .paragraphInput {
        margin: 10px;
        width: 100%;
        border: 2px solid;
        padding: 0 10px 0 10px;
        font-size: 24px;
        height: 120px;
        overflow: hidden;
        text-align: center;
    }

    .typingInput {
        width: 100%;
        border: 2px solid;
        padding: 10px;
        margin: 10px;
        font-size: 24px;
        text-align: center;
    }

    .paraWord {
        border-radius: 10px;
        margin: 1px;
    }

    .error {
        background-color: indianred;
    }

    .correct {
        background-color: lightgreen;
    }

    .current {
        background-color: teal;
    }

    .modalContent {
        padding: 20px;
    }

    .modalRefreshButton {
        width: 100%;
        background-color: #ddd;
        border-radius: 3px;
        padding: 2%;
        text-align: center;
        text-decoration: none;
    }

    .duringRestart {
        background-color: #ddd;
        padding: 5px;
        border-radius: 2px;
        display: inline-block;
    }
</style>
