<script lang="ts" setup>
definePageMeta({
  layout: 'none',
})

const isStarted = ref(false)
const questions = ref<any[]>([])
const chosenAnswers = ref<any[]>([])
const currentQuestionIndex = ref(0)
const difficulties = [
  {
    name: 'Any Difficulty',
    value: 'any',
  },
  {
    name: 'Easy',
    value: 'easy',
  },
  {
    name: 'Medium',
    value: 'medium',
  },
  {
    name: 'Hard',
    value: 'hard',
  },
]
const categories = ref<any[]>([])

const selectedDifficulty = ref('any')
const selectedCategory = ref('any')

function categoryClass(value: any) {
  return {
    'bg-purple-600 hover:bg-purple-400 text-white': selectedCategory.value === value,
    'bg-white hover:bg-purple-600 text-black hover:text-white': selectedCategory.value !== value,
  }
}
function difficultyClass(value: any) {
  return {
    'bg-purple-600 hover:bg-purple-400 text-white': selectedDifficulty.value === value,
    'bg-white hover:bg-purple-600 text-black hover:text-white': selectedDifficulty.value !== value,
  }
}

function startQuiz() {
  const url = generateUrl()

  isStarted.value = true

  fetch(url)
    .then(response => response.json())
    .then((responseJson) => {
      // If results not returned successfully
      if (responseJson.response_code !== 0)
        return Promise.reject(responseJson)

      populateQuestions(responseJson.results)
    })
    .catch((error) => {
      console.error(error)
      // eslint-disable-next-line no-alert
      alert(
        'Sorry, something went wrong trying to load the questions. Please try again.',
      )
    })
}

function generateUrl() {
  const difficultyParam
        = selectedDifficulty.value === 'any'
          ? ''
          : `&difficulty=${selectedDifficulty.value}`
  const categoryParam
        = selectedCategory.value === 'any' ? '' : `&category=${selectedCategory.value}`

  return `https://opentdb.com/api.php?amount=5${categoryParam}${difficultyParam}`
}

function shuffle(arr: any) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

function populateQuestions(responseJson: any) {
  if (responseJson.length > 0) {
    responseJson.forEach((questionData: any) => {
      const newQuestion = {} as any
      const answers = [] as any[]

      ;[...questionData.incorrect_answers, questionData.correct_answer].forEach((answer) => {
        const answerObject = { text: '', correct: false }

        ;[answerObject.text, answerObject.correct] = [answer, answer === questionData.correct_answer]

        answers.push(answerObject)
      })

      newQuestion.text = questionData.question
      newQuestion.answers = shuffle(answers)
      questions.value.push(newQuestion)
    })
  }
}
function selectAnswer(index: number): void {
  chosenAnswers.value[currentQuestionIndex.value] = index
}

function prevQuestion() {
  if (currentQuestionIndex.value > 0 && currentQuestionIndex.value > 0)
    currentQuestionIndex.value -= 1
}

function nextQuestion() {
  if (currentQuestionIndex.value < questions.value.length)
    currentQuestionIndex.value += 1
}

function selectQuestion(index: number) {
  if (chosenAnswers.value.length >= index)
    currentQuestionIndex.value = index
}

function calcScore() {
  let total = 0

  chosenAnswers.value.forEach((answer, index) => {
    if (
      typeof questions.value[index].answers[chosenAnswers.value[index]] !== 'undefined'
            && questions.value[index].answers[chosenAnswers.value[index]].correct
    )
      total += 1
  })

  return total
}

function completionMessage() {
  let text = 'Better luck next time'

  if (calcScore() >= questions.value.length * 0.8)
    text = 'Great work!'

  else if (calcScore() >= questions.value.length / 2)
    text = 'Good work!'

  return text
}

function answerCorrect(question: any, answerIndex: number) {
  return question.answers[answerIndex].correct
  // question.answers.find(a => a.correct == true);
}

function reset() {
  currentQuestionIndex.value = 0
  chosenAnswers.value = []
}

function resetQuestions() {
  questions.value = []
  startQuiz()
  reset()
}

function resetAll() {
  isStarted.value = false
  selectedCategory.value = 'any'
  selectedDifficulty.value = 'any'
  reset()
}

function decodeHtml(html: string) {
  const textArea = document.createElement('textarea')
  textArea.innerHTML = html
  return textArea.value
}

onMounted(async () => {
  const response = await fetch('https://opentdb.com/api_category.php')
  const data = await response.json()
  categories.value = data.trivia_categories.map((category: any) => ({
    name: category.name.replace(/\w+: /gi, ''),
    value: category.id,
  }))
})
</script>

<template>
  <Loading v-if="categories.length === 0 || (isStarted && questions.length === 0)" :text="categories.length === 0 ? 'Getting Ready!' : 'Thinking about questions!'" />
  <div v-if="categories.length > 0 && !isStarted" class="grid items-center justify-center text-center w-8/12 mx-auto gap-y-10 mt-20">
    <h1 class="text-6xl font-bold">
      NuxtQuiz
    </h1>
    <p class="text-2xl font-semibold">
      Difficulty
    </p>
    <div class="border-t-4 w-2/12 mx-auto" />
    <div class="grid grid-cols-4 justify-between gap-5 gap-x-5 gap-y-3">
      <button v-for="difficulty in difficulties" :key="difficulty.name" class="h-[55px] px-5" :class="difficultyClass(difficulty.value)" @click="selectedDifficulty = difficulty.value">
        {{ difficulty.name }}
      </button>
    </div>
    <p class="text-2xl font-semibold">
      Category
    </p>
    <div class="border-t-4 w-2/12 mx-auto" />
    <div class="grid grid-cols-5 w-full justify-between gap-x-5 gap-y-3">
      <button class="w-full h-[55px]" :class="categoryClass('any')" @click="selectedCategory = 'any'">
        Any Category
      </button>
      <button v-for="category in categories" :key="category.name" class="h-[55px] w-full px-5" :class="categoryClass(category.value)" @click="selectedCategory = category.value">
        {{ category.name }}
      </button>
    </div>
    <button class="h-[55px] px-10 block mx-auto bg-pink-500 hover:bg-pink-300" @click="startQuiz">
      Start
    </button>
  </div>
  <div v-if="currentQuestionIndex < questions.length && isStarted" :key="currentQuestionIndex" class="grid items-center justify-center text-center w-8/12 mx-auto gap-y-10 mt-40">
    <div class="flex w-full items-center justify-center text-center">
      <h2 class="text-6xl">
        <!-- eslint-disable-next-line vue/no-deprecated-filter -->
        {{ decodeHtml(questions[currentQuestionIndex].text) }}
      </h2>
    </div>
    <div class="grid gap-y-5 w-8/12 mx-auto">
      <button v-for="(answer, index) in questions[currentQuestionIndex].answers" :key="index" class="w-full h-[55px] px-5 hover:bg-purple-600 hover:text-white" :class="chosenAnswers[currentQuestionIndex] === index ? 'bg-purple-500  text-white' : 'bg-white text-black'" @click="selectAnswer(index)">
        {{ decodeHtml(answer.text) }}
      </button>
    </div>
    <div class="flex w-1/2 mx-auto items-center justify-between text-center">
      <button class="h-[55px] px-5 disabled:bg-gray-500 disabled:text-black bg-white text-black hover:bg-pink-500 hover:text-white" :disabled="currentQuestionIndex === 0" @click="prevQuestion">
        Back
      </button>
      <button class="h-[55px] px-5 disabled:bg-gray-500 disabled:text-black bg-pink-500 text-white hover:bg-pink-300" :disabled="chosenAnswers[currentQuestionIndex] == null || currentQuestionIndex >= questions.length" @click="nextQuestion">
        {{ currentQuestionIndex === questions.length - 1 ? 'Submit' : 'Next' }}
      </button>
    </div>
    <div class="flex gap-5 justify-center items-center text-center">
      <button v-for="(question, index) in questions" :key="index" class="h-[55px] w-[55px] disabled:bg-gray-600 disabled:text-black" :class="index === currentQuestionIndex ? 'bg-pink-500 text-white' : 'bg-white text-black'" :disabled="chosenAnswers.length < index" @click="selectQuestion(index)">
        {{ index + 1 }}
      </button>
    </div>
  </div>
  <div v-else-if="currentQuestionIndex >= questions.length && questions.length > 0" class="grid items-center justify-center text-center w-8/12 mx-auto gap-y-10 mt-20">
    <div class="flex justify-center">
      <p class="flex justify-center items-center text-center border-4 h-[200px] w-[200px] rounded-full text-4xl">
        Score: {{ calcScore() }}/{{ questions.length }}
      </p>
    </div>
    <div>
      <h2 class="text-4xl text-center">
        {{ completionMessage() }}
      </h2>
    </div>
    <div class="border-t-4" />
    <div v-for="(question, index) in questions" :key="index" class="grid w-full gap-y-5 mb-5">
      <h3 class="text-center text-2xl">
        {{ decodeHtml(question.text) }}
      </h3>
      <div class="border-t-4 w-3/12 mx-auto" />
      <p
        class="flex bg-green-500 h-[55px] px-5 w-1/2 mx-auto text-center justify-center items-center"
      >
        Correct answer: {{ decodeHtml(question.answers.find(a => a.correct === true).text) }}
      </p>
      <p
        v-if="!answerCorrect(question, chosenAnswers[index])"
        class="flex bg-red-500 h-[55px] px-5 w-1/2 mx-auto text-center justify-center items-center"
      >
        Your answer: {{ decodeHtml(question.answers[chosenAnswers[index]].text) }}
      </p>
    </div>
    <div class="grid grid-cols-3 items-center mb-20 mx-auto w-8/12 gap-x-5">
      <button class="h-[55px] px-5 bg-white text-black hover:bg-pink-500 hover:text-black" @click="reset">
        Try Again
      </button>
      <button class="h-[55px] px-5 bg-white text-black hover:bg-pink-500 hover:text-black" @click="resetQuestions">
        Restart with new questions
      </button>
      <button class="h-[55px] px-5 bg-white text-black hover:bg-pink-500 hover:text-black" @click="resetAll">
        New Setup
      </button>
    </div>
  </div>
</template>
