<template>
  <div id="game-scene" :class="{'scene in': show,
              'scene out': !show}">
     <div class="card player a" :class="{'flipped': cardAFlipped,'out' : cardAOut, 'in': !cardAOut, 'selected': cardASelected, 'shake':cardAShake}" @click="cardAClick()">
       <div class="front face">
         <div class="power">{{userCardPower}}</div>
       </div>
       <div class="back face"></div>
     </div>

     <div class="card player b" :class="{'flipped': cardBFlipped,'out' : cardBOut, 'in': !cardBOut, 'selected': cardBSelected, 'shake':cardBShake}" @click="cardBClick()">
       <div class="front face"><div class="power">{{userCardPower}}</div></div>
       <div class="back face"></div>
     </div>

     <div class="card player c" :class="{'flipped': cardCFlipped,'out' : cardCOut, 'in': !cardCOut, 'selected': cardCSelected, 'shake':cardCShake}" @click="cardCClick()">
       <div class="front face"><div class="power">{{userCardPower}}</div></div>
       <div class="back face"></div>
     </div>

     <div class="energy-bg">
       <div class="energy opponent" :style="{'width':opponentEnergyWidth}"></div>
       <div class="energy player" :style="{'width':playerEnergyWidth}"></div>
     </div>
     <div class="battle-indicator"></div>

     <div class="card battle" :class="{'out':cardBattleOut, 'in':!cardBattleOut,'shake' :battleCardShake}">
      <div class="front face"><div class="power">{{battleCardPower}}</div></div> 
       <div class="back face"></div>
     </div>

     <div class="fire to-left" :class="{'attack' : fireAttack}"></div>
      <div class="fire to-right" :class="{'attack': fireAttack2Right}"></div>

  </div>
</template>

<script>
  import Constant from './constant'
  export default {
    data () {
      return {
        show: false,
        youWin: true,
        cardAFlipped: false,
        cardBFlipped: false,
        cardCFlipped: false,
        cardAOut: true,
        cardBOut: true,
        cardCOut: true,
        cardASelected: false,
        cardBSelected: false,
        cardCSelected: false,
        cardBattleOut:true,
        
        battleCardShake:false,

        battleCardObject: null,
        fireObject: null,
        fireObject2right:null,
        userCardObject:null,

        fireAttack: false,
        fireAttack2Right: false,

        transitionState: '',
        cardAShake:false,
        cardBShake:false,
        cardCShake:false,

        playerEnergy: 100,
        opponentEnergy: 100,

        opponentEnergyWidth: '210px',
        playerEnergyWidth: '210px',
        
        userCardPower: 0,
        battleCardPower: 100


      }
    },
    mounted () {
      Constant.Event.$on(Constant.MSG_START_SCENE, function () {
        this.show = true
        setTimeout(this.cardFlipped, 400)
        


      }.bind(this))
      this.battleCardObject = document.querySelector('.card.battle')
        this.fireObject = document.querySelector('.fire.to-left')
        this.fireObject2right = document.querySelector('.fire.to-right')
    },
    methods: {
      cardFlipped () {
        this.cardAFlipped = true
        this.cardBFlipped = true
        this.cardCFlipped = true

        this.cardAOut = false
        this.cardBOut = false
        this.cardCOut = false
      },
      showGameOverScene () {
        Constant.Event.$emit(Constant.MSG_OVER_GAME_SCENE, this.userWon)
      },
      
      cardAClick () {
        this.cardASelected = true
        this.cardAFlipped = false
        this.cardBOut = true
        this.cardCOut = true
       
         this.battle ()
      },
      cardBClick () {
        this.cardBSelected = true
        this.cardBFlipped = false
        this.cardAOut = true
        this.cardCOut = true
         this.battle ()
      },
      cardCClick () {
        this.cardCSelected = true
        this.cardCFlipped = false
        this.cardBOut = true
        this.cardAOut = true
        
         this.battle ()
      },

       randomizeEnergy () {
        this.userCardPower = Math.round(Math.random() * 60) + 40
        this.battleCardPower = Math.round(Math.random() * 60) + 40
        console.log('userCardPower ' + this.userCardPower)
        console.log('battleCardPower ' + this.battleCardPower)
      },

      battle (){
        this.randomizeEnergy ()
        this.handleTransitionEnd(this.battleCardObject)
        this.transitionState = Constant.OPPONENT_CARD_TRANSITION_END
        this.cardBattleOut = false
      },
      handleAnimationEnd (htmlObj) {
        var listener = function (e) {
          e.target.removeEventListener('webkitAnimationEnd', listener)
          this.handleTransitionEvent(e)
        }.bind(this)

        htmlObj.addEventListener('webkitAnimationEnd', listener)
      },
      handleTransitionEnd (htmlObj) {
        var listener = function (e) {
          e.target.removeEventListener('webkitTransitionEnd', listener)
          this.handleTransitionEvent(e)
        }.bind(this)

        htmlObj.addEventListener('webkitTransitionEnd', listener)
      },
      restart () {
        this.cardASelected = false
        this.cardBSelected = false
        this.cardCSelected = false 
        this.cardBattleOut = true 
        this.transitionState = Constant.OPPONENT_CARD_TRANSITION_END
        this.fireAttack = false 
        this.battleCardShake = false 
        this.fireAttack2Right = false 
        this.cardAShake = false
        this.cardBShake = false
        this.cardCShake = false

        this.cardFlipped()
      },
      handleTransitionEvent (e) {

         switch (this.transitionState) {
          case Constant.OPPONENT_CARD_TRANSITION_END:
            if (this.cardBattleOut === false) {
              this.transitionState = Constant.FIRE_TOWARD_LEFT_ANIMATION_END
              this.fireAttack = true
              this.handleAnimationEnd(this.fireObject)
            }
            break
          case Constant.FIRE_TOWARD_LEFT_ANIMATION_END:
            this.handleAnimationEnd(this.battleCardObject)
            this.battleCardShake = true
            this.transitionState = Constant.OPPONENT_CARD_ANIMATION_END
            this.hurtOpponent(this.userCardPower, this.battleCardPower)
            break

          case Constant.OPPONENT_CARD_ANIMATION_END:

            this.handleAnimationEnd(this.fireObject2right)
            this.fireAttack2Right = true
            this.transitionState = Constant.FIRE_TOWARD_RIGHT_ANIMATION_END
            break

          case Constant.FIRE_TOWARD_RIGHT_ANIMATION_END:
          var userCard = ''
            if (this.cardASelected) {
              this.cardAShake = true
              userCard = '.card.player.a'
            } else if (this.cardBSelected) {
              this.cardBShake = true
              userCard = '.card.player.b'
            } else if (this.cardCSelected) {
              this.cardCShake = true
              userCard = '.card.player.c'
            }
            this.userCardObject = document.querySelector(userCard)
            this.handleAnimationEnd(this.userCardObject)
            this.hurtPlayer(this.battleCardPower, this.userCardPower)
            this.transitionState = Constant.USER_CARD_SHAKE_END

            break
          case Constant.USER_CARD_SHAKE_END:
           if(this.gameOver()){
            if(this.playerWin()){
              this.youWin=true
            }
            else
              this.youWin = false
             this.showGameOverScene()
            }

          else{
            this.restart()
          }


          break
        

      }
      },
      gameOver () {
        return (this.playerEnergy <= 0 || this.opponentEnergy <= 0)
      },
      playerWin () {
        if (this.playerEnergy <= 0) {
          return false
        }

        return true
      },
     

      hurtPlayer (attackPower, defensePower) {
       
        var diff = attackPower - defensePower
        if (diff > 0) {
          this.playerEnergy = Math.max(this.playerEnergy - diff, 0)
          this.playerEnergyWidth = this.playerEnergy / 100 * 210 + 'px'
        }
       console.log('attack opponent,player energy:' + this.playerEnergy+' diff:'+diff+' playerEnergyWidth:' + this.playerEnergyWidth)
      },
      hurtOpponent (attackPower, defensePower) {
        var diff = attackPower - defensePower
        
        if (diff > 0) {
          this.opponentEnergy = Math.max(this.opponentEnergy - diff, 0)
          this.opponentEnergyWidth = this.opponentEnergy / 100 * 210 + 'px'
        }
       console.log('attack player,opponent energy:' + this.opponentEnergy+' diff:'+diff+ ' opponentEnergyWidth:'+this.opponentEnergyWidth)
      }
    }
    

  }
</script>

<style scoped>
  #game-scene {
    background: url(../../static/images/battle-bg.png) no-repeat;
  }

  .card {
    position: absolute;
    width: 140px;
    height: 180px;
    perspective: 700;
    transition: all .5s ease-out;

    bottom: 250px;
  }

  .card.a {left: 15px;}
  .card.b {left: 170px;}
  .card.c {left: 325px;}

  .card > .face {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border-radius: 4px;
    backface-visibility: hidden;
    transition: all 0.5s ease-out;
  }

  .card > .face.front {
    background: #81d1e9 url(../../static/images/front-face.png);
    transform: rotate3d(0, 1, 0, 0deg);
  }

  .card > .face.back {
    background: #4474b5 url(../../static/images/back-face-pattern.png);
    transform: rotate3d(0, 1, 0, 180deg);
    border: 3px solid white;
  }

  .card.flipped > .face.front {
    transform: rotate3d(0, 1, 0, -180deg);
  }

  .card.flipped > .face.back {
    transform: rotate3d(0, 1, 0, 0deg);
  }

  .card.player.out {
    transform: translate3d(0, 300%, 0);
  }

  .card.player.in {
    transform: translate3d(0, 0, 0);
  }
  .card.battle {
    bottom: 250px;
  }
  .card.battle.out {
    left: -200px;
  }

 .card.battle.in {
    
    left: 40px;
  }




  .card.player.flipped {
    cursor: pointer;
  }

  .card.player.in.flipped:hover {
    transform: translate3d(0, -10px, 0);
  }

  .card.selected {
    bottom: 250px;
    left: 300px;
  }

  @keyframes shake {
    0%   {transform: translate3d(0, 0, 0);}
    20%  {transform: translate3d(5%, 0, 0);}
    40%  {transform: translate3d(-5%, 0, 0);}
    60%  {transform: translate3d(5%, 0, 0);}
    80%  {transform: translate3d(-5%, 0, 0);}
    100% {transform: translate3d(0, 0, 0);}
  }

  .card.shake{animation: shake 800ms ease-out}

  .fire {
    position: absolute;
    bottom: 300px;
    width: 50px;
    height: 50px;
    opacity: 0;
    animation-timing-function: ease-out;
    animation-duration: 400ms;
  }

  .fire.to-left {background-image: url(../../static/images/fire-left.png);}
  .fire.to-right {background-image: url(../../static/images/fire-right.png);}

  @keyframes fire-2-left {
  0%, 20% {opacity: 1; transform: translate3d(300px, 0, 0)}
  60% {transform: translate3d(100px, 0, 0);}
  100% {opacity: 0; transform: translate3d(100px, 0, 0);}
  }

  @keyframes fire-2-right {
  0%, 20% {opacity: 1; transform: translate3d(130px, 0, 0)}
  60% {transform: translate3d(330px, 0, 0);}
  100% {opacity: 0; transform: translate3d(330px, 0, 0);}
  }

  .fire.to-left.attack {animation-name: fire-2-left;}
  .fire.to-right.attack {animation-name: fire-2-right;}


.battle-indicator {
    position: absolute;
    width: 80px;
    height: 80px;
    top: 0;
    left: 200px;
    background: url(../../static/images/battle.png)
  }
  .energy-bg {
    border-bottom: 1px solid #333;
    background: #ababab;
    height: 30px;
  }
  .energy {
    position: absolute;
    width: 210px;
    height: 30px;
    transition: all .3s ease-out;
  }

  .energy.opponent {
    background: url(../../static/images/blue-hp.png) repeat;
    left: 0;
  }
  .energy.player {
    background: url(../../static/images/red-hp.png) repeat;
    right: 0;
  }

  .card .power {
    position: absolute;
    width: 100%;
    text-align: center;
    bottom: 30px;
    font-size: 2em;
    color: #fff
  }
</style>
