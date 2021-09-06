<template>
    <va-card tag='b' style="margin: 70px; padding: 40px" color="#fbfbfb">
        <h1 style="font-size: 25px;">5S / SOSSP Database</h1><br>
        <va-input
            class="mb-4"
            label="Speed / Period"
            placeholder="Enter Spaceship Speed / Oscillator Period"
            v-model="speed"
            :rules="[value => /^(P?[0-9]+|(\([0-9]+,[0-9]+\)|[0-9]*)c(\/[0-9]+[od]?)?)$/.test(value)]"
        />
        <va-button @click="readDatabase"> Confirm </va-button><br><br><br><br>
        <code style="background: #f0f0f0; padding: 10px; display: block; white-space: pre-wrap; font-weight: normal">
            {{ output }}
        </code>

        <va-modal v-model="showModal" :message="message" :title="title" />
    </va-card>
</template>

<script>
import oscillators from '!raw-loader!./assets/sossp.sss.txt'
import oblique from '!raw-loader!./assets/oblique.sss.txt'
import diagonal from '!raw-loader!./assets/diagonal.sss.txt'
import orthogonal from '!raw-loader!./assets/orthogonal.sss.txt'
export default {
    name: 'App',
    components: {},
    data() {
        return {
            speed: "",
            output: "Waiting user input...",
            showModal: false,
            title: "",
            message: ""
        }
    },
    methods: {
        readDatabase() {
            // Error Handling
            if (!/^(P?[0-9]+|(\([0-9]+,[0-9]+\)|[0-9]*)c(\/[0-9]+[od]?)?)$/.test(this.speed)) {
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

            let db = ""
            if (dx === 0 && dy === 0) db = oscillators
            else if (dy === 0) db = orthogonal
            else if (dx === dy) db = diagonal
            else db = oblique

            // Iterate through
            for (let x of db.split("\n")) {
                tokens = x.split(", ")
                if (parseInt(tokens[2]) === dx && parseInt(tokens[3]) === dy && parseInt(tokens[4]) === p) {
                    this.output = `#C Population: ${tokens[0]}\nx = 0, y = 0, rule = ${tokens[1]}\n${tokens[5]}`
                    return
                }
            }

            if (dx === 0 && dy === 0) this.output = `Oscillator of period ${p} not found in database!`
            else this.output = `Spaceship of speed ${this.speed} not found in database!`
        }
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
