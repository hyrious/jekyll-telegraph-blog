---
title: Bookmarks
---
<script>
const fk = () => {
  const face = ['(',')','[',']','|','|']
  const eyes = ['￣','￣','—','—','＿','＿','﹁','﹁','^','^','~','~','=','=','@','@','·','·','‵','′','>','<','→','←','〒','〒','┬','┬','T','T','゜','゜','。','。','●','●','◎','◎','⊙','⊙','○','○','╯','╰','︶','︶','〃','〃','Φ','Φ','Θ','Θ','+','+','x','x','$','$','≧','≦','ｕ','ｕ','Ｕ','Ｕ']
  const mouth = ['—','-','_','。','，','.','o','O','3','ε','ｪ','v','u','_',',','ω','Д','﹏','ヘ','︶','︿','▽','△','∀','<','>','○','ロ','□','Ｑ','＠','皿','◇','︹','＾','﹌','c','ｰ','C','c','t','(○○)','(工)','-','ゝ','(··)']
  const blah = ["''",'b','||','#','╬','＊','//',';;','::','メ','〆','z']
  const lHand = ['\\','n','m','v','w','<','m@','o','O','○','~','凸','╭∩╮','d','ㄏ','╰','╭','つ','ゞ','ﾉ','┌','ψ','彡','=','—', '～','︿','っ','づ','┗','┍']
  const rHand = ['/','n','m','v','y','w','>','@m','☞','o','O','○','~','凸','╭∩╮','b','ㄟ','╮','╯','ﾉ','↗','↘','つ','σ','ゞ',
        'ﾉ','┘','ψ','彡','=','—','～','︿','っ','づ','┛','┑']

  const rand = (l = 1) => Math.random() * l
  let faceRnd  = 0.65
  let mouthRnd = 0.8
  let blahRnd  = 0.2
  let lHandRnd = 0.4
  let rHandRnd = 0.4
  let faceLen  = face.length  / 2
  let eyeLen   = eyes.length  / 2
  let mouthLen = mouth.length / 2
  let blahLen  = blah.length  / 2
  let lHandLen = lHand.length / 2
  let rHandLen = rHand.length / 2
  let rLen, rObj

  rObj = rand()
  rLen = Math.floor(rand(faceLen)) * 2
  let _left_face = rObj < faceRnd ? face[rLen] : ''
  let _right_face = rObj < faceRnd ? face[rLen + 1] : ''

  rLen = Math.floor(rand(eyeLen)) * 2
  let _left_eye = eyes[rLen]
  let _right_eye = eyes[rLen + 1]

  rObj = rand()
  rLen = Math.floor(rand(mouthLen))
  let _mouth = rObj < mouthRnd ? mouth[rLen] : ''

  rObj = rand()
  rLen = Math.floor(rand(blahLen))
  let _blah = rObj < blahRnd ? blah[rLen] : ''

  rObj = rand()
  rLen = Math.floor(rand(lHandLen))
  let _lHand = rObj < lHandRnd ? lHand[rLen] : ''

  rObj = rand()
  rLen = Math.floor(rand(rHandLen))
  let _rHand = rObj < rHandRnd ? rHand[rLen] : ''

  return _lHand + _left_face + _left_eye + _mouth + _right_eye + _blah + _right_face + _rHand
}
</script>
<style>
  main a[href] {
    display: inline-block; border-bottom-width: 0;
    margin: 2px 0; padding: 0 2px; border-radius: 3px;
    border: .1em dashed rgba(0,0,0,0.4)
  }
  main a[href]:hover { background-color: rgba(0,0,0,0.7); color: #FFF }
  main a[href]:active { background-color: #000 }
  #fk { border: 1px solid transparent }
  #fk:hover { border: 1px solid grey }
  #fk:active { border: 1px solid #000 }
</style>

{% for link in site.bookmarks %}
[{{ link[0] }}]({{ link[1] }}){% endfor %}

[![][1]][2]

-fk: <span id="fk" style="-webkit-user-select: none; user-select: none; cursor: pointer; font-family: sans-serif;"></span>

<script>
  const updateFK = () => (document.getElementById('fk').innerText = fk())
  document.getElementById('fk').onclick = updateFK
  updateFK()
</script>

[1]: https://lemmmy.pw/osusig/sig.php?colour=hexff66aa&uname=hyrious&pp=0&countryrank&flagstroke
[2]: https://osu.ppy.sh/users/2899180
