<template>
</template>

<script>
  import Vue from 'vue'
  export default {
    MSG_START_SCENE: 'msg-start-scene',
    MSG_OVER_GAME_SCENE: 'msg-over-game-scene',

    Event: new Vue(),
    OPPONENT_CARD_TRANSITION_END: 'opponent_card_transition_end',
  	FIRE_TOWARD_LEFT_ANIMATION_END: 'fire_left_animation_end',
  	OPPONENT_CARD_ANIMATION_END: 'opponent_card_animation_end',
    FIRE_TOWARD_RIGHT_ANIMATION_END: 'fire_toward_right_animation_end',
    USER_CARD_SHAKE_END: 'user_card_shake_end'
  }
</script>

<style>
</style>
