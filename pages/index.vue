<template>
  <v-row justify="center" >
    <v-col cols="6">
      <v-card>
        <v-card-text>
          <p>カード</p>
          <div>
            <p class="d-flex align-center" v-for="(card, i) in hands" :key="i">
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
        {mark: 'S', val: '1'},
        {mark: 'H', val: '1',},
        {mark: 'D', val: '12'}, 
        {mark: 'S', val: '12'}, 
        {mark: 'S', val: '13'}
      ])
    this.getRole(this.hands)
  },
  data: ()=>({
    deck: [],
    hands: [],
    hasJoker: false,
    role: '',
  }),
  filters: {
    replace1toA(num){
      return num === 1 ? 'A' : num
    }
  },
  methods: {
    getCards(n){
      this.clearRole()
      this.clearHands();
      this.clearJokerState();
      const hand = [];
      for(let i = 0; i < n; i++){
        const l = this.deck.length;
        if(l === 0) this.addDeck();
        const index = Math.floor(Math.random() * l);
        hand.push(this.deck.splice(index, 1)[0])
      }
      this.hands.push(...hand);
      this.checkJoker()
      this.getRole(this.hands)
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
   getRole(hand){
     const s = new Set(hand.map(el => el.val).filter(el => el != 'Joker'))
     console.log(s)
     switch(s.size){
       case 1:
         this.role = 'FiveCard'
         break
      case 2:
        if(this.hasJoker){
          this.role = this.getNumberOfDuplication(hand, hand[0].val) === 2 ? 'FullHouse' : 'FourCard'
        }else{
          this.role = this.isFourCard(hand) ? 'FourCard' : 'FullHouse'
        }
        break
      case 3:
        if(this.hasJoker){
          this.role = 'ThreeCard'
        }else{
          this.role = this.isThreeCard(hand, s) ? 'ThreeCard' : 'TwoPair'
        }
        break
      case 4:
        if(this.hasJoker){
          const role = this.getOtherRole(hand) 
          this.role = role === 'NoPaier' ? 'OnePaier' : role
        }else{
          this.role = 'OnePair'
        }
        break
      case 5:
        this.role = this.getOtherRole(hand)
        break
     }
   },


   // カードの重複を数える関数
   getNumberOfDuplication(arr, num){
     return arr.filter(el => el.val == num).length
   },

    // フォーカード
    // handの１枚目の重複が無いか４の時
    isFourCard(hand){
      const duplicate = this.getNumberOfDuplication(hand, hand[0].val)
      return duplicate === 1 || duplicate == 4
    },

    // スリーカード
    isThreeCard(hand, set){
      return [...set].map((el) => this.getNumberOfDuplication(hand, el)).find(el => el === 3)
    },

    // ストレート系のロールを取得する
    getOtherRole(hand){
      const sortedHand = hand.map(el => Number(el.val)).sort((a, b) => a > b ? -1 : 1)
      const flash = this.isFlash(hand)
      const straight = this.isStraight(sortedHand)
      if(flash && straight) return 'StraighFlash'
      if(straight) return 'Straight'
      if(flash) return this.isRoyalStraightFlash(hand) ? 'RoyalStraightFlash' :  'Flash'
      if(flash === false && straight === false) return  'NoPaier'
    },
    
    // ストレートは渡ってきたハンドをソートして隣り合う要素の差がすべて１の時
    // つまり隣合うカードの差の合計が4の時
    // ジョーカーありの場合も隣合うカードの差の合計が４ならストレートが成立する
    isStraight(sortedHand){
      let totalDiff = 0
      console.log(sortedHand)
      for(let i = 0; i < sortedHand.length - 1; i++){
        totalDiff += sortedHand[i] - sortedHand[i + 1]
      }
      return totalDiff == 4 ? true : false
    },

    // フラッシュはハンドのマークだけのセットを作ってサイズが1の時
    isFlash(hand){
      return new Set(hand.map(el => el.mark)).size == 1 ? true : false
    },

   // ロイヤルストレートフラッシュはフラッシュの判定時に呼ばれる
   // handに１が含まれていたら14に変換してisStraightを呼ぶ
    isRoyalStraightFlash(hand){
      return this.isStraight(hand.map(el => el.val === '1' ? 14 : Number(el.val)).sort((a, b) => a > b ? -1 : 1))
    },
    
    isTwoPare(hand){},
    isOnePare(hand){},
    isFullhouse(hand){},
    isStraightFlash(hand){},
    isFiveCard(hand){},
  }
}
</script>
