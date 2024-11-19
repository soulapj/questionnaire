<template>
  <nav>
    <router-link to="/app">Nouvellepage</router-link>
  </nav>
  <Gamescore :scoreJoueur="this.scoreJoueur" :scoreCpu="this.scoreCpu"/>
  <h2 v-html="question"></h2>
  <div class="container" v-for="(answer, index) in this.answers" :key="index">
    <input
      type="radio"
      name="opt"
      v-model="chosenAnswer"
      :value="answer"
      :disabled="disabledRadio"
      :checked="checkRadio"
    />
    <label for="opt" v-html="answer"></label>
  </div>
  <button @click="submitAnswer">Valider</button>
  <p>{{ statusAnswer }}</p>
  <button @click="newQuestion">Nouvelle Question</button>
</template>

<script>
import Gamescore from './components/GameScore.vue';
export default {
  name: 'App',
  data() {
    return {
      question: '',
      incorrectAnswers: [],
      correctAnswer: '',
      chosenAnswer: '',
      scoreJoueur: '',
      scoreCpu: '',
      statusAnswer: '',
      disabledRadio: false,
      checkRadio: false,
    };
  },
  components: {
    Gamescore
  },
  methods: {
    newQuestion() {
      this.axios.get('https://opentdb.com/api.php?amount=1').then((res) => {
        console.log(res.data.results[0]);
        this.question = res.data.results[0].question;
        this.incorrectAnswers = res.data.results[0].incorrect_answers;
        this.correctAnswer = res.data.results[0].correct_answer;
      });
      this.statusAnswer = '';
      this.chosenAnswer = '';
      this.disabledRadio = false;
    },
    submitAnswer() {
      if (this.chosenAnswer == this.correctAnswer) {
        this.scoreJoueur++;
        this.statusAnswer = '👍👍👍 Bravo !';
        this.disabledRadio = true;
      } else {
        this.scoreCpu++;
        this.statusAnswer = `❌❌❌Dommage ! La bonne réponse était "${this.correctAnswer}!"`;
        this.disabledRadio = true;
      }

      setTimeout(() => {
        this.newQuestion();
      }, 4000);
    },
    toLocalStorage() {
      localStorage.setItem('scoreJoueur', this.scoreJoueur);
      localStorage.setItem('scoreCpu', this.scoreCpu);
    },
  },
  computed: {
    answers() {
      let answers = [...this.incorrectAnswers];
      answers.splice(
        Math.round(Math.random() * answers.length),
        0,
        this.correctAnswer
      );
      // answers.push(this.correctAnswer);
      // answer.sort(() => Math.random() - 0.5);
      return answers;
    },
  },
  watch: {
    scoreCpu: {
      handler() {
        this.toLocalStorage();
      },
    },
    scoreJoueur: {
      handler() {
        this.toLocalStorage();
      },
    },
  },
  created() {
    this.newQuestion();
    localStorage.getItem('scoreJoueur')
      ? (this.scoreJoueur = localStorage.getItem('scoreJoueur'))
      : (this.scoreJoueur = 0);
    localStorage.getItem('scoreCpu')
      ? (this.scoreCpu = localStorage.getItem('scoreCpu'))
      : (this.scoreCpu = 0);
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
button {
  margin-top: 20px;
  padding: 10px;
  background-color: rgb(81, 156, 85);
  color: white;
  border: none;
  border-radius: 3px;
}
.container {
  display: flex;
  margin: 10px;
  align-items: center;
  justify-content: space-around;
  width: auto;
}
</style>
