<script>
import { getFirestore, collection, getDocs, addDoc} from "firebase/firestore";
import app from './firebaseIntegration.js'
const db = getFirestore(app)
console.log("Got Here")

let firebaseCategories = []

const querySnapshots = await getDocs(collection(db, "listOfCollections"));
querySnapshots.forEach((doc) => {
  firebaseCategories.push(doc.data().collectionName)
  console.log(firebaseCategories[0])
});

export default {
    data() {
        return {
            selectedCategory: 0,
            categories: firebaseCategories,
            question: [],
            answer: [],
            count: 0,
            countArray: [],
            degrees: "rotateY(0deg)",
            flipped: false,
            newQuestion: "",
            newAnswer: "",
        }
    },
    async create() {
        console.log("Creating Method")
        this.setAnswersQuestions()
    },
    methods: {
        increaseCount() {
            if (this.count < this.question.length-1) {
                this.count++
            }
        },
        createCountArray() {
            this.countArray = []
            for (let i=0; i < this.question.length; i++) {
                this.countArray.push(i)
            }
            return this.countArray
        },
        decreaseCount() {
            if (this.count != 0) {
                this.count--
            }
        },
        rotateCard() {
            if (this.flipped) {
                this.degrees = "rotateY(0deg)"
            } else {
                this.degrees = "rotateY(180deg)"
            }
            this.flipped = !this.flipped
        },
        resetCard() {
            this.flipped = false
            this.degrees = "rotateY(0deg)"
        },
        nextQuestion() {
            this.resetCard()
            this.increaseCount()
        },
        async inputNewQA() {
            if (this.newQuestion && this.newAnswer) {
                const docRef = await addDoc(collection(db, "ITIL"), {
                    question: this.newQuestion,
                    answer: this.newAnswer
                });
                this.question.push(this.newQuestion)
                this.answer.push(this.newAnswer)
                this.clearTextArea()
                this.createCountArray()
            }
        },
        async getCategoryAnswerDocs(category) {
            let computed_answers = []
            const querySnapshot = await getDocs(collection(db, category));
            querySnapshot.forEach((doc) => {
                this.answer.push(doc.data().answer)
            });
        },
        async getCategoryQuestionDocs(category) {
            let computed_question = []
            const querySnapshot = await getDocs(collection(db, category));
            querySnapshot.forEach((doc) => {
                this.question.push(doc.data().question)
            });
        },
        clearTextArea() {
            this.newAnswer = ""
            this.newQuestion = ""
        },
        previousQuestion() {
            this.resetCard()
            this.decreaseCount()
        },
        getNonRepeatingNumber(previousNumbers) {
            let good = false
            let j = 0
            console.log(previousNumbers)
            while (!good) {
                j = Math.floor(Math.random() * (this.countArray.length))
                let results = false
                for (let i=0; i < previousNumbers.length; i++) {
                    if (j == previousNumbers[i]) {
                        results = true
                        break
                    } else {
                        results = false
                    }
                }
                if (!results) {
                    return j
                }
            }
        },
        shuffleCards() {
            let ranNums = []
            this.createCountArray()
            let i = this.countArray.length
            let j = 0
            let previousNumbers = []
            while (i--) {
                j = this.getNonRepeatingNumber(previousNumbers)
                ranNums.push(this.countArray[j])
                previousNumbers.push(j)
            }
            console.log(ranNums)
            this.countArray = ranNums
            console.log(this.countArray)
            this.count = 0
        },
        async setAnswersQuestions() {
            await this.getCategoryAnswerDocs(this.categories[this.selectedCategory])
            await this.getCategoryQuestionDocs(this.categories[this.selectedCategory])
        }
    },
    async mounted() {
        await this.setAnswersQuestions()
        this.createCountArray()
    },
    computed: {
        getQuestion() {
            console.log(this.question)
            if (this.question.length > 0 && this.count < this.question.length) {
                return this.question[this.countArray[this.count]]
            } else {
                return 'No More Questions'
            }
        },
        getAnswer() {
            console.log(this.answer)
            if (this.answer.length > 0 && this.count < this.answer.length) {
                return this.answer[this.countArray[this.count]]
            } else {
                return 'No More Answers'
            }
        }
    }
}
</script>

<template>
    <div class="flipcard">
        <div class="flip-card-inner" @click="rotateCard" v-bind:style="{transform: degrees}">
            <div class="flip-card-front">
                <p id="current-question">{{ getQuestion }}</p>
            </div>
            <div class="flip-card-back">
                <p id="current-answer"> {{ getAnswer }}</p>
            </div>    
        </div>
    </div>
    <button id="next-question" @click="nextQuestion">Next Question</button>
    <button id="shuffle-cards" @click="shuffleCards">Shuffle Cards</button>
    <button id="previous-question" @click="previousQuestion">Previous Question</button>

    <div class="submit-new-qa">
        <div class="new-question-div">
            <label for="question-label">Input New Question</label>
            <br/>
            <textarea type="text" rows="10" v-model="newQuestion" cols="35" name="question-input" id="question-input"></textarea>
        </div>
        <div class="new-answer-div">
            <label for="answer-input">Input New Answer</label>
            <br/>
            <textarea type="text" rows="10" v-model="newAnswer" cols="35" name="answer-input" id="answer-input"></textarea>
        </div>
        <button id="submit-new-answer" @click="inputNewQA">Submit New Answer</button>
    </div>
</template>