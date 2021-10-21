<template>
  <div>
    <ScoreBoard
      :playerScore="this.playerScore"
      :computerScore="this.computerScore"
    />

    <template v-if="this.question">
      <h1 v-html="this.question"></h1>

      <template v-for="(question, index) in this.answers" v-bind:key="index">
        <input
          v-bind:disabled="this.answerSubmitted"
          type="radio"
          name="options"
          :value="question"
          v-model="this.chosenAnswer"
        />

        <label v-html="question"></label><br />
      </template>

      <button
        v-if="!this.answerSubmitted"
        v-on:click="submitAnswer()"
        class="send"
        type="button"
      >
        Send
      </button>

      <section v-if="this.answerSubmitted" class="result">
        <template v-if="this.chosenAnswer == this.correctAnswer">
          <h4>✔️ Parabéns, a resposta está correta.</h4>
        </template>
        <template v-else>
          <h4
            v-html="
              '❌ Que pena, a resposta está errada. A resposta correta seria: &#34;' +
              this.correctAnswer +
              '&#34;.'
            "
          ></h4>
        </template>

        <button v-on:click="this.getNewQuestion()" class="send" type="button">
          Next Question
        </button>
      </section>
    </template>
  </div>
</template>

<script>
import ScoreBoard from "@/components/ScoreBoard.vue";

export default {
  name: "App",
  components: {
    ScoreBoard,
  },
  data() {
    return {
      question: undefined,
      incorrectAnswers: undefined,
      correctAnswer: undefined,
      chosenAnswer: undefined,
      answerSubmitted: false,
      playerScore: 0,
      computerScore: 0,
    };
  },
  computed: {
    answers: function () {
      var answers = JSON.parse(JSON.stringify(this.incorrectAnswers));
      answers.splice(
        Math.round(Math.random() * answers.length),
        0,
        this.correctAnswer
      );
      return answers;
    },
  },
  methods: {
    submitAnswer: function () {
      if (!this.chosenAnswer) {
        alert("Pick one of options!");
      } else {
        this.answerSubmitted = true;

        if (this.chosenAnswer == this.correctAnswer) {
          this.playerScore++;
          localStorage.setItem("playerScore", String(this.playerScore));
        } else {
          this.computerScore++;
          localStorage.setItem("computerScore", String(this.computerScore));
        }
      }
    },
    getNewQuestion: function () {
      this.answerSubmitted = false;
      this.chosenAnswer = undefined;

      this.axios
        .get(
          "https://opentdb.com/api.php?amount=1&category=18&difficulty=easy&type=multiple"
        )
        .then((response) => {
          let data = response.data.results[0];

          this.question = data.question;
          this.correctAnswer = data.correct_answer;
          this.incorrectAnswers = data.incorrect_answers;
        });
    },
  },
  created() {
    let computerScore = localStorage.getItem("computerScore");
    let playerScore = localStorage.getItem("playerScore");

    if (computerScore != null) {
      this.computerScore = Number(computerScore);
    }

    if (playerScore != null) {
      this.playerScore = Number(playerScore);
    }

    this.getNewQuestion();
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1 {
  margin-top: 40px;
}

input[type="radio"] {
  margin: 12px 4px;
}

button.send {
  margin-top: 12px;
  height: 40px;
  min-width: 120px;
  padding: 0 16px;
  color: #fff;
  background-color: #1867c0;
  border: 1px solid #1867c0;
  cursor: pointer;
}
</style>
