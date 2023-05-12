<script setup>
import { ref,computed } from 'vue'
import axios from 'axios';



const diff = ref(true)
let edd = 1
let questions = ref([null])
let easy = async (edd) => {
	await axios.get(`https://opentdb.com/api.php?amount=10&category=18&${ edd !== 1 ? `difficulty=${ edd }` :''}&type=multiple`)
		.then(r => {
			questions.value = r.data.results

		})
		.catch(err => console.log(err))
	diff.value = false
}


const quizCompleted = ref(false)
const currentQuestion = ref(0)
const score = computed(() => {
	let value = 0
	questions.value.map(q => {
		if(q.selected != null && q.correct_answer == q.selected) {
			value++
		}
	})
	return value
})

const getCurrentQuestion = computed(() => {
	let question = questions.value[currentQuestion.value]
	question.index = currentQuestion.value
	question.incorrect_answers=[...question.incorrect_answers,question.correct_answer]
	question.correct_answer = 3
	return question
})

const SetAnswer = (e) => {
	questions.value[currentQuestion.value].selected = e.target.value
	e.target.value = null
}

const NextQuestion = () => {
	if(currentQuestion.value < questions.value.length - 1) {
		currentQuestion.value++
		return
	}

	quizCompleted.value = true
}
</script>

<template>
	<main class="app">
		<h1>The Quiz</h1>
		<div class="diff" v-if="diff">
			<form action="" method="get" @submit.prevent="easy(edd)">
				<label for="" class="option2"> Select Difficulty: </label>
				<select v-model="edd" name="" id="">
					<option value="1" selected>Any Difficulty</option>
					<option value="easy">Easy </option>
					<option value="medium">Medium </option>
					<option value="hard"> Hard</option>
				</select>
				<button type="submit"> start </button>
			</form>
		</div>
		<section class="quiz" v-else-if="!quizCompleted">
			<div class="quiz-info">
				<span class="question">{{ getCurrentQuestion.question }}</span>
				<span class="score">Score {{ score }}/{{ questions.length }}</span>
			</div>
			<div class="incorrect_answers">
				<label v-for="(option, index) in getCurrentQuestion.incorrect_answers" :key="'option' + index" :class="`option ${getCurrentQuestion.selected == index
						? index == getCurrentQuestion.correct_answer
							? 'correct'
							: 'wrong'
						: ''
					} ${getCurrentQuestion.selected != null &&
						index != getCurrentQuestion.selected
						? 'disabled'
						: ''
					}`">

					<input type="radio" :id="'option' + index" :name="getCurrentQuestion.index" :value="index"
						v-model="getCurrentQuestion.selected" :disabled="getCurrentQuestion.selected" @change="SetAnswer" />
					<span>{{ option }}</span>
				</label>
			</div>
			<button @click="NextQuestion" :disabled="!getCurrentQuestion.selected"> {{ getCurrentQuestion.index ==
				questions.length - 1 ? 'Finish' : getCurrentQuestion.selected == null ? 'Select an option' : 'Next question' }}
			</button>
		</section>
		<section v-else>
			
			<h2>You have finished the quiz!</h2>
			<h3>{{ score > 5 ? 'Congratulations you success in the quiz' : 'Unfortunately you Fail in the quiz' }}</h3>
			<p>Your score is {{ score }}/{{ questions.length }}</p>
		</section>
	</main>
</template>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Montserrat', sans-serif;
}

body {
	background-color: #271c36;
	color: #FFF;
}

.app {
	display: flex;
	flex-direction: column;
	align-items: center;
	padding: 2rem;
	height: 100vh;
}

h1 {
	font-size: 2rem;
	margin-bottom: 2rem;
}

.quiz {
	background-color: #382a4b;
	padding: 1rem;
	width: 100%;
	max-width: 870px;
}

.quiz-info {
	display: flex;
	justify-content: space-between;
	margin-bottom: 1rem;
}

.quiz-info .question {
	color: #8F8F8F;
	font-size: 1.25rem;
}

.quiz-info.score {
	color: #FFF;
	font-size: 1.25rem;
}

.incorrect_answers {
	margin-bottom: 1rem;
}

.option ,select,.option2{
	padding: 1rem;
	display: block;
	background-color: #271c36;
	margin-bottom: 0.5rem;
	border-radius: 0.5rem;
	cursor: pointer;
}
select{
	color: white;
	border: none;
	outline: none;
	width: 500px;
	background-color: #2d213f;
	margin-bottom: 70px;
}
.option:hover {
	background-color: #2d213f;
}

.option.correct {
	background-color: #2cce7d;
}

.option.wrong {
	background-color: #ff5a5f;
}

.option:last-of-type {
	margin-bottom: 0;
}

.option.disabled {
	opacity: 0.5;
}

.option input {
	display: none;
}

button {
	appearance: none;
	outline: none;
	border: none;
	cursor: pointer;
	padding: 0.5rem 1rem;
	background-color: #2cce7d;
	color: #2d213f;
	font-weight: 700;
	text-transform: uppercase;
	font-size: 1.2rem;
	border-radius: 0.5rem;
}

button:disabled {
	opacity: 0.5;
}

h2 ,h3{
	font-size: 2rem;
	margin-bottom: 2rem;
	text-align: center;
}
h3{
	font-size: 1.5rem;
}
p {
	color: #8F8F8F;
	font-size: 1.5rem;
	text-align: center;
}
</style>