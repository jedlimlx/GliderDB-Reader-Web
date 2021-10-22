<template>
    <va-card tag='b' style="margin: 60px; padding: 40px" color="#fbfbfb">
        <h1 style="font-size: 25px;">5S / SOSSP Database</h1><br>
        <va-input
            class="mb-4"
            label="Speed / Period"
            placeholder="Enter Spaceship Speed / Oscillator Period"
            v-model="speed"
            :rules="[value => /^(P?[0-9]+|(\([0-9]+,[0-9]+\)|[0-9]*)c(\/[0-9]+[od]?)?)$/.test(value)]"
        />
        <va-button @click="read5SDatabase"> Confirm </va-button><br><br><br><br>
        <code style="background: #f0f0f0; padding: 10px; display: block; white-space: pre-wrap; font-weight: normal">
            {{ output }}
        </code>

        <va-modal v-model="showModal" :message="message" :title="title" />
    </va-card>
    <va-card tag='b' style="margin: 60px; padding: 40px" color="#fbfbfb">
        <h1 style="font-size: 25px;">GliderDB Database</h1><br>
        <va-input
            class="mb-4"
            label="Speed / Period"
            placeholder="Enter Spaceship Speed / Oscillator Period"
            v-model="speed"
            :rules="[value => /^(P?[0-9]+|(\([0-9]+,[ ]*[0-9]+\)|[0-9]*)c(\/[0-9]+[od]?)?)$/.test(value)]"
        />
        <va-checkbox v-model="ignoreVelocity" :label=label1 /><br><br>
        <va-input
            class="mb-4"
            label="Rule"
            placeholder="Enter the rule to search in"
            v-model="rule"
        />
        <va-checkbox v-model="ignoreRule" :label=label2 /><br><br>
        <va-button @click="readGliderDB"> Confirm </va-button><br><br><br><br>

        <va-list-item v-for="(glider, index) in gliders" :key="index" style="width:100%">
            <code
                style="background: #f0f0f0; padding: 10px; display: block;
                white-space: pre-wrap; font-weight: normal; width:100%"
            >
                {{ glider.rle }}
            </code>
        </va-list-item>
    </va-card>
</template>

<script>
import oscillators from '!raw-loader!./assets/sossp.sss.txt'
import oblique from '!raw-loader!./assets/oblique.sss.txt'
import diagonal from '!raw-loader!./assets/diagonal.sss.txt'
import orthogonal from '!raw-loader!./assets/orthogonal.sss.txt'

import r1_moore_gliders from '!raw-loader!./assets/R1-C2-NM-gliders.db.txt'
import r1_moore_oscillators from '!raw-loader!./assets/R1-C2-NM-oscillators.db.txt'
import r1_c3_moore_gliders from '!raw-loader!./assets/R1-C3-NM-gliders.db.txt'
import r2_moore_gliders from '!raw-loader!./assets/R1-C3-NM-gliders.db.txt'

import r1_neumann_gliders from '!raw-loader!./assets/R1-C2-NN-gliders.db.txt'
import r2_neumann_gliders from '!raw-loader!./assets/R1-C3-NM-gliders.db.txt'

export default {
    name: 'App',
    components: {},
    data() {
        return {
            speed: "",
            rule: "",
            output: "Awaiting user input...",  // For 5S
            gliders: [  // For GliderDB
                { rle: "Awaiting user input..." }
            ],
            showModal: false,
            title: "",
            message: "",
            label1: "Ignore Velocity",
            label2: "Ignore Rule",
            ignoreVelocity: false,
            ignoreRule: false
        }
    },
    methods: {
        insertNewLine(string, maxLength){
            let newString = []
            for (let i = 0; i < string.length; i++) {
                newString.push(string[i])
                if (i % maxLength === 0 && i !== 0) {
                    newString.push("\n")
                }
            }

            return newString.join("")
        },
        isSupersetOf(setB, setA) {
            if (typeof(setB) === 'undefined') return true
            if (typeof(setA) === 'undefined') return false

            let result = true
            setA.forEach(ele => {
                if (!(setB.has(ele))) result = false
            })

            return result
        },
        parseSpeed() {
            // Error Handling
            if (!/^(P?[0-9]+|(\([0-9]+,[ ]*[0-9]+\)|[0-9]*)c(\/[0-9]+[od]?)?)$/.test(this.speed)) {
                this.title = "Error"
                this.message = `"${this.speed}" is not a valid speed`
                this.showModal = !this.showModal
                return
            }

            let dx = 0, dy = 0, p = 1
            let tokens = this.speed.split("c")

            if (tokens.length === 1) {
                p = parseInt(tokens[0].replace("P", ""))
            } else if (tokens[1].length === 0) {
                dx = tokens[0].length > 0 ? parseInt(tokens[0]) : 1
                p = 1
            } else {
                const tokens2 = tokens[0].replace(/[()]/, "").split(",")
                if (tokens2.length === 1) dx = tokens[0].length > 0 ? parseInt(tokens[0]) : 1
                else {
                    dx = parseInt(tokens2[0])
                    dy = parseInt(tokens2[1])

                    // Ensure dx > dy
                    if (dx < dy) {
                        const temp = dx
                        dx = dy
                        dy = temp
                    }
                }

                p = parseInt(tokens[1].replace(/[/od]/, ""))
            }

            // Check for diagonal speeds
            if (this.speed[this.speed.length - 1] === 'd') dy = dx
            return [dx, dy, p]
        },
        parseRule(rule) {
            let range = 1, numStates = 2, neighbourhood = "M", birth, survival;
            if (/^[Bb][0-8]*\/[Ss][0-8]*[VH]?$/.test(rule)) {
                let tokens = rule.split("/")
                birth = new Set(Array.from(tokens[0].replace(/[BbSs]/, "")).map(x => parseInt(x)))
                survival = new Set(Array.from(tokens[1].replace(/[BbSs]/, "")).map(x => parseInt(x)))

                switch (rule[rule.length - 1]) {
                    case "V":
                        neighbourhood = "N"
                        break
                    case "H":
                        neighbourhood = "H"
                        break
                    default: neighbourhood = "M"
                }
            } else if (/^[0-8]*\/[0-8]*\/[0-9]+[VH]?$/.test(rule)) {
                let tokens = rule.split("/")
                survival = new Set(Array.from(tokens[0]).map(x => parseInt(x)))
                birth = new Set(Array.from(tokens[1]).map(x => parseInt(x)))
                numStates = parseInt(tokens[2])

                switch (rule[rule.length - 1]) {
                    case "V":
                        neighbourhood = "N"
                        break
                    case "H":
                        neighbourhood = "H"
                        break
                    default: neighbourhood = "M"
                }
            }

            return [range, numStates, neighbourhood, birth, survival]
        },
        read5SDatabase() {
            const speedList = this.parseSpeed()
            const dx = speedList[0], dy = speedList[1], p = speedList[2]

            let db = ""
            if (dx === 0 && dy === 0) db = oscillators
            else if (dy === 0) db = orthogonal
            else if (dx === dy) db = diagonal
            else db = oblique

            // Iterate through
            let tokens
            for (let x of db.split("\n")) {
                tokens = x.split(", ")
                if (parseInt(tokens[2]) === dx && parseInt(tokens[3]) === dy && parseInt(tokens[4]) === p) {
                    this.output = `#C Population: ${tokens[0]}\nx = 0, y = 0, rule = ${tokens[1]}\n${tokens[5]}`
                    return
                }
            }

            if (dx === 0 && dy === 0) this.output = `Oscillator of period ${p} not found in database!`
            else this.output = `Spaceship of speed ${this.speed} not found in database!`
        },
        readGliderDB() {
            const parsedSpeed = this.parseSpeed()
            const dx = parsedSpeed[0], dy = parsedSpeed[1], p = parsedSpeed[2]

            const parsedRule = this.parseRule(this.rule)
            const range = parsedRule[0], numStates = parsedRule[1], neighbourhood = parsedRule[2],
                birth = parsedRule[3], survival = parsedRule[4]

            // Choose the correct database
            let database_name = ""
            let db_lookup = {
                "R1-C2-NM-gliders": r1_moore_gliders,
                "R1-C2-NM-oscillators": r1_moore_oscillators,
                "R1-C3-NM-gliders": r1_c3_moore_gliders,
                "R2-C2-NM-gliders": r2_moore_gliders,
                "R1-C2-NN-gliders": r1_neumann_gliders,
                "R2-C2-NN-gliders": r2_neumann_gliders
            }
            if (dx === 0 && dy === 0) database_name = `R${range}-C${numStates}-N${neighbourhood}-oscillators`
            else database_name = `R${range}-C${numStates}-N${neighbourhood}-gliders`

            let db = db_lookup[database_name]

            // Linear Search
            console.log(database_name)
            let tokens, minRule, maxRule
            let entries = db.split("\n")
            this.gliders = []
            for (let i = 0; i < entries.length; i++) {
                tokens = entries[i].split(":")

                // Pre-processing
                tokens[5] = Math.abs(parseInt(tokens[5]))
                tokens[6] = Math.abs(parseInt(tokens[6]))
                if (tokens[4].includes("/")) {
                    tokens[4] = parseInt(tokens[4].split("/")[0])
                } else tokens[4] = parseInt(tokens[4])

                // Checking speed is correct
                if (((tokens[4] === p && ((tokens[5] === Math.abs(dx) && tokens[6] === Math.abs(dy)) ||
                    (tokens[5] === Math.abs(dy) && tokens[6] === Math.abs(dx))))) || this.ignoreVelocity) {
                    minRule = this.parseRule(tokens[2])
                    maxRule = this.parseRule(tokens[3])

                    // Checking minimum and maximum rules
                    if ((this.isSupersetOf(birth, minRule[3]) && this.isSupersetOf(survival, minRule[4]) &&
                        this.isSupersetOf(maxRule[3], birth) && this.isSupersetOf(maxRule[4], survival))
                        || this.ignoreRule) {
                        console.log(birth, minRule[3], maxRule[3])
                        this.gliders.push({
                            rle: (dx === 0 && dy === 0 ? `#C P${tokens[4]}\n` :
                                    `#C (${tokens[5]}, ${tokens[6]})/${tokens[4]}\n`) +
                                (tokens[0] !== "" ? `#C Name: ${tokens[0]}\n` : '') +
                                (tokens[1] !== "" ? `#C Discovered by: ${tokens[1]}\n` : '') +
                                `#C Min Rule: ${tokens[2]}\n` +
                                `#C Max Rule: ${tokens[3]}\n` +
                                `x = ${tokens[7]}, y = ${tokens[8]}, rule = ${this.rule}\n` +
                                this.insertNewLine(tokens[9], 100)
                        })
                    }
                }
            }

            if (this.gliders.length === 0) {
                if (dx === 0 && dy === 0) this.gliders = [ { rle: `Spaceship not found in database!` } ]
                else this.gliders = [ { rle: `Spaceship not found in database!` } ]
            }
        }
    },
    computed: {
        console: () => console
    }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*text-align: center;*/
  color: #2c3e50;
  margin-top: 20px;
}
</style>
