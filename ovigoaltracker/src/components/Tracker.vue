    <template>
        <div id="tracker" class="tracker-container">
            <h1 class="title">Has Ovi Broken the Goals Record?</h1>
            <div v-if="loading">Loading...</div>
            <div v-else-if="error">Fucky-wucky UwU: {{ error }}</div>
            <div v-else>
                <div v-if="recordBroken">
                    <h1 class="headline">Yes, Alex Ovechkin is currently the NHL's all time goals leader.</h1>
                    <h2 class="subheader">To date, he has scored {{ goals }} goals, which is {{ goalDelta }} more than Wayne Gretzky's 894.</h2>
                    <div v-if="isActive">
                        <p class="details">He's still playing in the NHL, and is scoring at a rate of {{ currentScoringRate }} {{ goalRatePlural }} per game.</p>
                    </div>
                </div>
                <div v-else>
                    <h1 class="headline">Nope, he hasn't broken it yet.</h1>
                    <h2 class="subheader">He's currently sitting at {{ goals }} career goals.</h2>
                    <div v-if="isActive">
                        <p class="details">He needs {{ goalDelta }} more {{ goalDeltaPlural }} to tie the record, and {{ goalsToBreak }} {{ goalBreakPlural }} to break it.</p>
                        <p class="details">Ovi is currently scoring at a pace of {{ currentScoringRate }} {{ goalRatePlural }} per game.</p>
                        <p class="details">At his current pace, he should tie the record in {{ gamesToTie }} {{ gamesTiePlural }}, and break it in {{ gamesToBreak }} {{ gamesBreakPlural }}.</p>
                    </div>
                    <div v-else>
                        <p class="details">Ovechkin is not currently an active NHL player. He is currently {{ goalDelta }} goals short of Gretzky's record.</p>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <style scoped>
    .tracker-container {
        display:flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;

    }

    .title {
        font-family: 'Roboto Mono';
        font-size: 3.5rem;
        font-weight: bold;
        margin-bottom: 0rem;
        margin-top: 0rem;
    }

    .headline {
        font-family: 'Roboto Mono';
        margin-top: 0rem;
        font-size: 2.5rem;
        font-weight: bold;
    }

    .subheader {
        font-family: 'Roboto Mono';
        margin-top:0rem;
        margin-bottom:0rem;
        font-size: 1.8rem;
        font-weight: bold;
    }
    .details {
        font-family: 'Roboto Mono';
        font-size: 1.2rem;
        line-height: 1.6;
        margin-bottom: 1rem;
    }

    /* Responsive scaling for different screen sizes */
    @media (max-width: 768px) {
        .headline {
            font-size: 2rem;
        }

        .subheadline {
            font-size: 1.5rem;
        }

        .details {
            font-size: 1rem;
        }
    }
    </style>

    <script>
    import { ref, onMounted, computed } from 'vue';

    export default {
        name: 'App',

        setup() {

            const goals = ref(null);
            const loading = ref(true);
            const error = ref(null);
            const goalDelta = ref(null)
            const goalsToBreak = ref(null)
            const currentScoringRate = ref(0);
            const gamesToTie = ref(null);
            const gamesToBreak = ref(null);
            const recordBroken = ref(null);
            const isActive = ref(null);
            const goalRatePlural = computed(() => (currentScoringRate.value === 1 ? 'goal' : 'goals'));
            const goalDeltaPlural = computed(() => (goalDelta.value === 1 ? 'goal' : 'goals'));
            const goalBreakPlural = computed(() => (goalsToBreak.value === 1 ? 'goal' : 'goals'));
            const gamesTiePlural = computed(() => (gamesToTie.value === 1 ? 'game' : 'games'));
            const gamesBreakPlural = computed(() => (gamesToBreak.value === 1 ? 'game' : 'games'));

            
            const fetchGoals = async () => {
                
                try {
                    
                    // make the request to nhl api. We'll just hardcode this to go to ovi.
                    const response = await fetch('/api/v1/player/8471214/landing')
                    if (!response.ok) {
                        throw new Error('Failed to fetch data from NHL api');
                    }
                    
                    const data = await response.json();
                    console.log(data);
                    isActive.value = data.isActive;
                    const careerRegSeason = data.careerTotals?.regularSeason;
                    
                    // Figure out his current goal scoring rate using data from his last 5 games
                    // This isn't super accurate, and is likely to vary wildly from game to game as
                    // his scoring rate changes, but it's fine for a rough ballpark.
                    // A better method would be to look at shooting rates and shooting percentages, 
                    // ideally broken down by opponent, and utilize that to estimate how many goals 
                    // he will score in the upcoming games. This would be a good use case for
                    // machine learning, but is way beyond the scope of this project. 
                    
                    const last5Games = data.last5Games;
                    let goalsLast5Games = 0
                    for (let i = 0; i < last5Games.length; i++) {
                        goalsLast5Games += last5Games[i].goals;
                    }
                    if (goalsLast5Games != 0) {
                        currentScoringRate.value = goalsLast5Games / 5;
                    } else {
                        // If he hasn't scored a goal in his last 5 games, we'll default to his career scoring rate.
                        currentScoringRate.value = careerRegSeason.goals / careerRegSeason.gamesPlayed;
                    }
                    
                    goals.value = careerRegSeason.goals ?? 0;
                    goalDelta.value = Math.abs(894 - goals.value); 
                    recordBroken.value = goals.value > 894;
                    goalsToBreak.value = goalDelta.value + 1;
                    
                    // Round games up to the nearest whole game.
                    gamesToTie.value = Math.ceil(goalDelta.value / currentScoringRate.value);
                    gamesToBreak.value = Math.ceil(goalsToBreak.value / currentScoringRate.value);
                    
                } catch (e) {
                    error.value = e.message;
                } finally {
                    loading.value = false;
                }

            };
            
            onMounted(fetchGoals);   
            
            return {goals, loading, error, goalDelta, goalsToBreak, goalRatePlural, goalDeltaPlural, goalBreakPlural, currentScoringRate, gamesToTie, gamesToBreak, recordBroken, isActive, gamesBreakPlural, gamesTiePlural };
        },
    };
    </script>