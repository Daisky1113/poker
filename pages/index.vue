<template>
  <v-row justify="center" >
    <v-col cols="6">
      <v-card>
        <v-card-text>
          <p>カード</p>
          <div>
            <p class="d-flex align-center" v-for="(card, i) in sorteadHand" :key="i">
              <span class="mr-3"><v-img max-height="20" max-width="20" :src="`/img/${card.mark}.png`" alt=""></v-img></span>
              <span class="subtitle-2">{{ card.val |  replace1toA}}</span>
            </p>
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="getCards(5)">カード配布</v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
    <v-col cols="6" sm="8" md="6">
      <v-card>
        <v-card-text>
          <p>役</p>
          <p>{{ role }}</p>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>

export default {
  components: {
  },
  created(){
    this.addDeck()
    //--------------------------------
    // debug
    //---------------------------------
    this.hands.push(...[
        {mark: 'S', val: 1},
        {mark: 'S', val: 13,},
        {mark: 'S', val: 12}, 
        {mark: 'S', val: 11}, 
        {mark: 'S', val: 10}, 
        // {mark: 'Joker', val: 'Joker'}
      ])
    this.checkJoker()
    this.setRole(this.getRole(this.hands))
  },
  data: ()=>({
    deck: [],
    hands: [],
    hasJoker: false,
    role: '',
  }),
  computed: {
    sorteadHand(){
      this.hands.sort((a, b) => a.val < b.val ? -1 : 1)
    }
  },
  filters: {
    replace1toA(num){
      return num === 1 ? 'A' : num
    }
  },
  computed:{
    sorteadHand(){
      return this.hands.sort((a, b) => a.val < b.val ? -1 : 1)
    }
  },
  methods: {
    getCards(n){
      this.clearRole()
      this.clearHands();
      this.clearJokerState();
      this.getRandomCard(n)
      this.checkJoker()
      this.setRole(this.getRole(this.hands))
    },

    getRandomCard(n){
      const hand = [];
      for(let i = 0; i < n; i++){
        if(this.deck.length == 0) this.addDeck()
        const l = this.deck.length
        const index = Math.floor(Math.random() * l);
        hand.push(this.deck.splice(index, 1)[0])
      }
      this.hands.push(...hand);
    },

    addDeck(){
      const marks = ['S', 'H', 'D', 'C']
      marks.forEach(el => {
        for(let i = 0; i < 13; i++){
          this.deck.push({
            mark: el,
            val: i + 1
          })
        }
      });

      this.deck.push({
        mark: 'Joker',
        val: 'Joker'
      })
    },

    clearHands(){
      this.hands.splice(0, 5)
    },
    checkJoker(){
      const jokerIndex = this.hands.findIndex(el => el.mark == 'Joker')
      if(jokerIndex != -1){
        this.hasJoker = true
      }
    },
    clearJokerState(){
      this.hasJoker = false
    },
    judge(){
      this.checkJoker()
    },

    /*
     手役判定関数
     ジョーカーは予め除外した手配を関数に渡す
     まずhandのsetを作ってサイズによって行う処理を振り分ける
     1. fiveCard
     2. fourCard fullHouse fourCard(joker)
     3. threeCard twoPair threeCard(joker)
     4. onePair 
     5. straight flash straightFlash RoyalStraightFlash NoPair onePair(Joker)
    */
   clearRole(){
     this.role = ''
   },
   setRole(role){
     this.role = role
   },

   getRole(hand){
     const sorteadHand = hand.filter(el => el.val != 'Joker').map(el => Number(el.val)).sort((a, b) => a > b? -1 : 1)
     const s = new Set(sorteadHand)

     switch(s.size){
      case 1:
         return 'FiveCard'
      case 2:
        return this.isFourCard(sorteadHand, s) ? 'FourCard' : 'FullHouse'
      case 3:
        return this.isThreeCard(sorteadHand, s, this.hasJoker) ? 'ThreeCard' : 'TwoPaier'
      case 4:
      case 5:
        const flash = this.isFlash(hand)
        const straight = this.isStraight(sorteadHand, this.hasJoker)
        const aStraight = this.isAStraight(sorteadHand)
        if(flash && aStraight) return 'RoyalStraightFlash'
        if(flash && straight) return 'StraighFlash'
        if(straight || aStraight ) return 'Straight'
        if(flash) return  'Flash'
        if(this.isOnePare(sorteadHand, s, this.hasJoker, straight, flash)) return 'OnePair'
        return 'NoPair'
     }
   },


   // カードの重複を数える関数
   getNumberOfDuplication(arr, num){
     return arr.filter(el => el == num).length
   },

    // フォーカード
    // 重複している数字のどちらかの要素数が１の時
    isFourCard(hand, set){
      return [...set].map(el => this.getNumberOfDuplication(hand, el)).find(el => el === 1) ? true: false
    },

    // スリーカード
    // 重複している要素のどれかが３つあった時
    // ジョーカーありの時は問答無用でthreecard
    isThreeCard(hand, set, hasJoker){
      if(hasJoker) return true
      return [...set].map((el) => this.getNumberOfDuplication(hand, el)).find(el => el === 3)
    },
    
    // ストレートは渡ってきたハンドをソートして隣り合う要素の差がすべて１の時
    // つまり隣合うカードの差の合計が4の時
    // ジョーカーありの場合も隣合うカードの差の合計が４ならストレートが成立する
    isStraight(sortedHand, hasJoker){
      console.log(sortedHand)
      let totalDiff = 0
      for(let i = 0; i < sortedHand.length - 1; i++){
        totalDiff += sortedHand[i] - sortedHand[i + 1]
      }

      if(hasJoker && totalDiff == 3){
        return true
      }

      return totalDiff == 4 ? true : false
    },

    // Aストレートを判断する
    // ロイヤルはこれとフラッシュの組み合わせでみる
    isAStraight(sortedHand){
      if(sortedHand.includes(1)){
        return this.isStraight(sortedHand.map(el => el === 1 ? 14 : el).sort((a, b) => a > b ? -1 : 1))
      }else{
        return false
      }
    },

    // フラッシュはハンドのマークだけのセットを作ってサイズが1の時
    isFlash(hand){
      return new Set(hand.map(el => el.mark)).size == 1 ? true : false
    },
    isOnePare(hand, s, hasJoker, isStraight, isFlash){
      if(s.size === 4 && hand.length === 5) return true
      return isStraight === false && isFlash === false && hasJoker ? true : false
    },

    isRoyalStraightFlash(hand){},
    isTwoPare(hand){},
    isFullhouse(hand){},
    isStraightFlash(hand){},
    isFiveCard(hand){},
  }
}
</script>
