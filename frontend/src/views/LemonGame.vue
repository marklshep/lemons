<template>
    <div>
        <header>
            <h1> Lemons </h1>
        </header>
        <div id="board">
            <div v-for="(lll, index) in letterStates" class="row">
                <button v-for="(ll, index) in lll"
                        :disabled="ll.order === -2"
                        @click="setSolution(ll)"
                        class="tile"
                        :class="getStatus(ll.order)"
                >
                    {{ll.letter}}
                </button>
                <!-- <button v-bind:class="getStatus(ll.order)" @click="setSolution(ll)" > {{ll.letter}} </button> -->
            </div>
            <button @click="enterSolution"> Enter </button>
            <div id="answers">
                {{solution}}
                <button v-for="word in solutionWords"> {{word}} <b>X </b> </button>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import dict from '!raw-loader!../assets/dict.txt';

interface LetterState {
    letter: string;
    order: number;
}

@Component
export default class LemonGame extends Vue {
    private something: string = 'something';
    private ughList: string[] = ['something1', 'soemthign2'];
    private solution: string = '';
    private solutionWords: string[] = [];
    private order: number = 0;
    private lemonLetters: string[][] = [
        ['L', 'W', 'U', 'G'],
        ['N', 'E', 'M', 'T'],
        ['E', 'R', 'S', 'C'],
        ['U', 'T', 'O', 'E'],
    ];
    private letterStates: LetterState[][] = [];

    private dictSet: Set<string> = new Set<string>();

    private mounted(): void {
        for (const word of dict.match(/\b(\w+)\b/g)) {
            // console.log(word);
            this.dictSet.add(word);
        }
        for (const lll of this.lemonLetters) {
            const newLetters: LetterState[] = [];
            for (const ll of lll) {
                const ls: LetterState = {letter: ll, order: -1};
                newLetters.push(ls);
            }
            this.letterStates.push(newLetters);
        }
        console.log(this.letterStates);
    }

    private setSolution(ls: LetterState) {
        if (ls.order === -1) {
            this.solution = this.solution + ls.letter;
            ls.order = this.order;
            this.order += 1;
        } else {
            this.solution = this.solution.substring(0, ls.order) +
                            this.solution.substring(ls.order + 1, this.solution.length);
            ls.order = -1;
            this.order -= 1;
            for (const lll of this.letterStates) {
                for (const ll of lll) {
                    if (ll.order !== -1 && ll.order !== 0 && ll.order !== -2) {
                        ll.order -= 1;
                    }
                }
            }
        }
        if (this.dictSet.has(this.solution)) {
            console.log('that is a word');
        }
        console.log(this.dictSet.has(this.solution));
        console.log(this.order);
        console.log(this.letterStates);
    }
    private enterSolution() {
        if (this.dictSet.has(this.solution)) {
            this.solutionWords.push(this.solution);
            this.solution = '';
            for (const lll of this.letterStates) {
                for (const ll of lll) {
                    if (ll.order !== -1 && ll.order !== -2) {
                        ll.order = -2;
                    }
                }
            }
            this.order = 0;
        } else {
            console.log('not a solution');
        }
    }
    private getStatus(order: number): string {
        if (order === -1) {
            return 'absent';
        } else if (order === -2) {
            return 'correct';
        } else {
            return 'present';
        }
    }

}

</script>


<style scoped>
.bolded { font-weight: bold; }
#board {
    display: grid;
    grid-template-rows: repeat(6, 1fr);
    grid-gap: 5px;
    padding: 10px;
    box-sizing: border-box;
    --height: min(420px, calc(var(--vh, 100vh) - 310px));
    height: var(--height);
    width: min(350px, calc(var(--height) / 6 * 4));
    margin: 0px auto;
}
.row {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-gap: 5px;
}
.tile {
    width: 100%;
    font-size: 2rem;
    line-height: 2rem;
    font-weight: bold;
    vertical-align: middle;
    text-transform: uppercase;
    position: relative;
}
.correct {
    background-color: #6aaa64 !important;
}

.present {
    background-color: #c9b458 !important;
}

.absent {
    background-color: #787c7e !important;
}
#answers {
    height: 174px;
}
h1 {
    margin: 4px 0;
    font-size: 36px;
}

header {
    border-bottom: 1px solid #ccc;
    margin-bottom: 30px;
    position: relative;
}
body {
    font-family: 'Clear Sans', 'Helvetica Neue', Arial, sans-serif;
    text-align: center;
    max-width: 500px;
    margin: 0px auto;
}

</style>
