<template>
  <WindowFrame titleText="プレイヤーボックス画面" display-property="private.display.playerBoxWindow" align="center" fixSize="300, 400">
    <div class="contents">
      <select v-model="currentPlayerKey">
        <option v-for="player in players" :key="player.key" :value="player.key">{{player.name}}</option>
      </select>
      <fieldset v-if="currentPlayerKey">
        <legend>{{getPlayerName(currentPlayerKey)}}({{getType}})</legend>
        <label>
          チャット文字色
          <input
            type="color"
            :value="getFontColor"
            @change="event => changePlayerFontColor(event.target.value, event.target.nextElementSibling.firstElementChild.checked)"
          />
          <label>過去ログ反映<input type="checkbox" checked /></label>
        </label>
        <fieldset>
          <legend>接続(peerId)</legend>
          <ul>
            <li v-for="member in getMembers()" :key="member.peerId">{{member.peerId}}
              <span v-if="member.peerId === myPeerId">(この画面)</span>
            </li>
          </ul>
        </fieldset>
        <!-----------------
         ! マップ
         !---------------->
        <fieldset>
          <legend>マップ</legend>
          <ul class="objList">
            <li v-for="character in getMapObject('character', 'field')" :key="character.key">
              <CharacterChip :type="character.kind" :objKey="character.key" />
              <fieldset class="fontColorArea">
                <legend>チャット文字色</legend>
                <div>
                  <select
                    :value="character.fontColorType"
                    @change="event => changeFontColorType(character.key, Number(event.target.value))"
                  >
                    <option value="0">主と同じ</option>
                    <option value="1">個別</option>
                  </select>
                  <input
                    type="color"
                    :value="character.fontColorType === 0 ? getFontColor : character.fontColor"
                    @change="event => changeCharacterFontColor(character.key, event.target.value, event.target.parentNode.nextElementSibling.firstElementChild.checked)"
                    :disabled="character.fontColorType === 0"/>
                </div>
                <label>過去ログ反映<input type="checkbox" checked /></label>
              </fieldset>
            </li>
            <li v-for="mapMask in getMapObject('mapMask', 'field')" :key="mapMask.key">
              <MapMaskChip :type="mapMask.kind" :objKey="mapMask.key" />
            </li>
            <li v-for="chit in getMapObject('chit', 'field')" :key="chit.key">
              <ChitChip :type="chit.kind" :objKey="chit.key" />
            </li>
          </ul>
        </fieldset>
        <!-----------------
         ! キャラクター待合室
         !---------------->
        <fieldset v-if="myPlayerType === 'GM' || currentPlayerKey === selfPlayerKey">
          <legend>キャラクター待合室</legend>
          <ul class="objList">
            <li v-for="character in getMapObject('character', 'waiting')" :key="character.key">
              <CharacterChip :type="character.kind" :objKey="character.key" />
              <button @click="toMap(character.key)">マップへ</button>
            </li>
          </ul>
        </fieldset>
        <!-----------------
         ! 墓場
         !---------------->
        <fieldset v-if="myPlayerType === 'GM' || currentPlayerKey === selfPlayerKey">
          <legend>墓場</legend>
          <ul class="objList">
            <li v-for="character in getMapObject('character', 'graveyard')" :key="character.key">
              <CharacterChip :type="character.kind" :objKey="character.key" />
              <button @click="toMap(character.key)">マップへ</button>
            </li>
            <li v-for="mapMask in getMapObject('mapMask', 'graveyard')" :key="mapMask.key">
              <MapMaskChip :type="mapMask.kind" :objKey="mapMask.key" />
              <button @click="toMap(mapMask.key)">マップへ</button>
            </li>
            <li v-for="chit in getMapObject('chit', 'graveyard')" :key="chit.key">
              <ChitChip :type="chit.kind" :objKey="chit.key" />
              <button @click="toMap(chit.key)">マップへ</button>
            </li>
          </ul>
        </fieldset>
      </fieldset>
    </div>
  </WindowFrame>
</template>

<script>
import { mapState, mapActions, mapGetters } from 'vuex'
import WindowFrame from '../WindowFrame'
import WindowMixin from '../WindowMixin'
import CharacterChip from '../map/character/CharacterChip'

export default {
  name: 'playerBoxWindow',
  mixins: [WindowMixin],
  components: {
    WindowFrame,
    CharacterChip
  },
  data () {
    return {
      currentPlayerKey: null
    }
  },
  methods: {
    ...mapActions([
      'imageTagChange',
      'addImage',
      'windowClose',
      'emptyProperty',
      'setProperty',
      'getPieceObj',
      'changeChatFontColor',
      'changePieceInfo'
    ]),
    changeFontColorType (key, value) {
      const targetCharacter = this.characters.filter(character => character.key === key)[0]
      if (!targetCharacter) return
      const index = this.characters.indexOf(targetCharacter)
      this.setProperty({property: `public.character.list.${index}.fontColorType`, value: value, isNotice: true, logOff: false})
    },
    changeCharacterFontColor (key, value, historyChange) {
      this.changeChatFontColor({
        key: key,
        color: value,
        historyChange: historyChange
      })
    },
    changePlayerFontColor (value, historyChange) {
      const targetPlayer = this.getPlayer
      console.log(targetPlayer.name, targetPlayer.key)
      this.changeChatFontColor({
        key: targetPlayer.key,
        color: value,
        historyChange: historyChange
      })
    },
    getPlayerName (playerKey) {
      const player = this.getObj(playerKey)
      if (!player) return ''
      return player.name
    },
    toMap (key) {
      const kind = key.split('-')[0]
      this.changePieceInfo({ propName: kind, key: key, place: 'field', isNotice: true })
    }
  },
  watch: {
  },
  computed: mapState({
    ...mapGetters([
      'getObj'
    ]),
    members: state => state.public.room.members,
    players: state => state.public.player.list,
    myPeerId: state => state.private.self.peerId,
    selfPlayerKey: state => {
      const player = state.public.player.list.filter(player => player.name === state.private.self.playerName)[0]
      return player ? player.key : null
    },
    myPlayerType: state => state.private.self.playerType,
    characters: state => state.public.character.list,
    getPlayer () {
      return this.getObj(this.currentPlayerKey)
    },
    getType () {
      const player = this.getPlayer
      if (!player) return ''
      return player.type
    },
    getFontColor () {
      const player = this.getPlayer
      if (!player) return ''
      return player.fontColor
    },
    getMembers () {
      return function () {
        return this.members.filter(member => member.name === this.currentPlayerKey)
      }
    },
    getMapObject (state) {
      return function (kind, place) {
        return state.public[kind].list.filter(target => target.owner === this.currentPlayerKey && target.place === place)
      }
    }
  })
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.contents {
  position: absolute;
  height: 100%;
  width: 100%;
  overflow-y: scroll;

  font-size: 12px;
}
  fieldset {
    padding: 0 0.5rem 0.5rem;
  }
  ul {
    margin: 0;
    padding: 0 0.5rem;
  }
  li {
    position: relative;
  }
  ul.objList {
    list-style: none;
    padding-left: 0;
  }
  ul.objList > li {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
  }
  ul.objList > li .character {
    margin-top: 1em;
    margin-right: 0.5rem;
  }
  input[type=color] {
    width: 2rem;
    height: 1.5rem;
    padding: 0 0.2rem;
    box-sizing: border-box;
  }
  select {
    height: 1.5rem;
  }
  .fontColorArea div {
    display: flex;
    flex-direction: row;
  }
</style>
