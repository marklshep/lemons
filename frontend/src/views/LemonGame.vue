<template>
    <div>
        <Transition>
            <div class="message" v-if="message">
                {{ message }}
                <!-- <pre v-if="grid">{{ grid }}</pre> -->
            </div>
        </Transition>
        <header>
            <h1> Lemons </h1>
        </header>
        <div id="board">
            <div v-for="(lll, i) in letterStates" class="row">
                <button v-for="(ll, j) in lll"
                        :disabled="ll.order === -2"
                        @click="setSolution(ll, j, i)"
                        class="tile"
                        :class="getStatus(ll.order, j, i)"
                >
                    {{ll.letter}}
                </button>
            </div>
            <button @click="enterSolution"> Enter </button>
            <div id="answers">
                {{getSolution(solution)}}
                <button v-for="(word, index) in solutionWords"
                        @click="removeSolution(word, index)"
                > {{getSolution(word)}} <b>X </b> </button>
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

interface SolutionLetter {
    x: number;
    y: number;
    letter: string;
}

@Component
export default class LemonGame extends Vue {
    private solution: SolutionLetter[] = [];
    private solutionWords: SolutionLetter[][] = [];
    private order: number = 0;
    private message: string = '';
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
    // function shake() {
    //     shakeRowIndex = currentRowIndex
    //     setTimeout(() => {
    //         shakeRowIndex = -1
    //     }, 1000)
    //     }
    private showMessage(msg: string, time = 1000) {
        this.message = msg;
        if (time > 0) {
            setTimeout(() => {
                this.message = '';
            }, time);
        }
    }
    private getSolution(sol: SolutionLetter[]): string {
        let solString: string = '';
        // change to comprehension?
        for (const sl of sol) {
            solString = solString + sl.letter;
        }
        return solString;
    }
    private setSolution(ls: LetterState, x: number, y: number) {
        if (ls.order === -1) {
            const solLetter: SolutionLetter = {
                x,
                y,
                letter: ls.letter,
            };
            this.solution.push(solLetter);
            ls.order = this.order;
            this.order += 1;
        } else {
            this.solution.splice(ls.order, 1);
            // this.solution = this.solution.substring(0, ls.order) +
//                 this.solution.substring(ls.order + 1, this.solution.length);
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
        if (this.dictSet.has(this.getSolution(this.solution))) {
            console.log('that is a word');
        }
        console.log(this.order);
        console.log(this.letterStates);
    }
    private enterSolution() {
        if (this.solution.length < 3) {
            // shake()
            this.showMessage(`All words must be at least three letters long`);
            return;
        }
        if (this.dictSet.has(this.getSolution(this.solution))) {
            this.solutionWords.push(this.solution);
            this.solution = [];
            let won: boolean = true;
            for (const lll of this.letterStates) {
                for (const ll of lll) {
                    if (ll.order !== -1 && ll.order !== -2) {
                        ll.order = -2;
                    }
                    if (ll.order !== -2) {
                        won = false;
                    }
                }
            }
            this.order = 0;
            if (won) {
                this.showMessage(`woah cool`);
                navigator.clipboard.writeText('woah cool');
            }
        } else {
            // shake()
            this.showMessage(`Not in word list`);
        }
    }
    private removeSolution(sol: SolutionLetter[], index: number) {
        for (const sl of sol) {
            console.log('in remove');
            this.letterStates[sl.y][sl.x].order = -1;
        }
        this.solutionWords.splice(index, 1);
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
.message {
    position: absolute;
    left: 50%;
    top: 80px;
    color: #fff;
    background-color: rgba(0, 0, 0, 0.85);
    padding: 16px 20px;
    z-index: 2;
    border-radius: 4px;
    transform: translateX(-50%);
    transition: opacity 0.3s ease-out;
    font-weight: 600;
}
.message.v-leave-to {
    opacity: 0;
}
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
