Title: Three.js VR
Description: 가상현실을 Three.js로 사용하는 방법.
TOC: VR - 기본 사항

가상현실 앱을 three.js로 만드는 것은 매우 기본적으로 three.js에게 WedXR을 사용할 것이라 알리기만 하면된다. WedXR에 몇가지 사항을 명확하게 해야 한다. 이를 생각하여 보도록하자. 카메라가 가리키는 방향은 향하고 있는지 VR시스템에서 제공한다. 사용자가 머리를 돌려 보는 방향을 선택하기 때문이다. 비슷하게 각 시스템 이후 VR 시스템에서 시야와 화면비가 제공됩니다. (각 시스템은 시야와 디스플레이 측면이 다르다)

[반응형 웹페이지 만들기](threejs-responsive.html) 예시를 통하여 VR을 지원하도록 만들어 보겠습니다.

시작하기 전에 안드로이드 스마트폰, 구글 데이드림, 오큘러스 고, 오큘러스 리프트, 바이브,
삼성 기어 VR, [WebXR browser](https://apps.apple.com/us/app/webxr-viewer/id1295998056)가 설치된 아이폰과 같은 VR 지원 장치가 필요합니다.

다음으로, 로컬에서 실행 중인 경우 다음과 같은 간단한 웹 서버를 실행해야 합니다.
[the article on setting up](threejs-setup.html) 참조.

VR을 보는 데 사용하는 장치가 실행 중인 컴퓨터와 다른 경우
https를 통해 웹 페이지를 서비스해야 합니다. 그렇지 않으면 브라우저에서 사용을 허용하지 않습니다.
WebXR API. [the article on setting up](threejs-setup.html)에 언급된 서버
[Servez](https://greggman.github.io/servez)에는 https를 사용할 수 있는 옵션이 있습니다.
확인 후 서버를 시작합니다.

<div class="threejs_center"><img src="../resources/images/servez-https.png" class="nobg" style="width: 912px;"></div>

URL을 기록해 두십시오. 컴퓨터의 로컬 IP 주소가 필요합니다. 일반적으로 `192`, `172` 또는 `10`으로 시작합니다.  `https://` 부분을 포함한 전체 주소를 입력하세요.

VR 기기의 브라우저로 이동합니다. NOTE: 작업 컴퓨터와 VR 장치는 동일한 로컬 네트워크에 있어야 합니다.
또는 WiFi이고 아마도 홈 네트워크에 있어야 할것입니다. NOTE: 많은 카페에서 이러한 방법으로 기계 대 기계 연결.


아래와 같은 오류 메시지가 표시됩니다. "고급(advanced)"을 클릭한 다음 *진행(proceed)* 을 클릭한다.

<div class="threejs_center"><img src="resources/images/https-warning.gif"></div>

이제 예제를 실행할 수 있습니다.

실제로 WebVR 개발을 하려는 경우 배워야 할 또 다른 사항은 
[원격 디버깅(remote debugging)](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/) 이다.
이를 통해 콘솔 경보, 오류, 실제로 [코드 디버그(debug your code)](threejs-debugging-javascript.html)가  가능하다.

아래 코드가 작동하는 것을 보고 싶다면 이 사이트에서 코드를 실행할 수 있습니다.

가장 먼저 해야 할 일은 three.js를 포함시킨 후 VR 지원을 포함하는 것입니다.

```js
import * as THREE from './resources/three/r132/build/three.module.js';
+import {VRButton} from './resources/threejs/r132/examples/jsm/webxr/VRButton.js';
```

이후 three.js's WebXR 지원을 활성화 하며, 그것의 VR button을 페이지에 추가하여 주어야한다.

```js
function main() {
  const canvas = document.querySelector('#c');
  const renderer = new THREE.WebGLRenderer({canvas});
+  renderer.xr.enabled = true;
+  document.body.appendChild(VRButton.createButton(renderer));
```

three.js가 렌더 루프를 실행하도록 해야 합니다. 지금까지 우리는 
`requestAnimationFrame`loop 를 사용하였다. 하지만 VR을 지원하기 위해서 우리는 three.js가 우리의 render loop를 관리할 수 있도록 해야한다. 이 과정을 
`WebGLRenderer.setAnimationLoop`를 호출 및  루프를 호출하는 함수를 전달하여 진행 가능하다.

```js
function render(time) {
  time *= 0.001;

  if (resizeRendererToDisplaySize(renderer)) {
    const canvas = renderer.domElement;
    camera.aspect = canvas.clientWidth / canvas.clientHeight;
    camera.updateProjectionMatrix();
  }

  cubes.forEach((cube, ndx) => {
    const speed = 1 + ndx * .1;
    const rot = time * speed;
    cube.rotation.x = rot;
    cube.rotation.y = rot;
  });

  renderer.render(scene, camera);

-  requestAnimationFrame(render);
}

-requestAnimationFrame(render);
+renderer.setAnimationLoop(render);
```

세부 사항이 하나 더 있습니다. 우리는 아마 카메라 높이를 설정해야 합니다.
예로는 서 있는 사용자의 평균키 입니다.

```js
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
+camera.position.set(0, 1.6, 0);
```

큐브를 카메라 앞으로 이동합니다.

```js
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

cube.position.x = x;
+cube.position.y = 1.6;
+cube.position.z = -2;
```

카메라가  `z = 0` 에 있고 카메라는 -z축을 바라보기 때문에 우리는  `z = -2`에  배치 시켰습니다(cude)


이것은 매우 중요한 점을 제시합니다. **Units in VR are in meters**.
다른 말로  **One Unit = One Meter**. 이것은 카메라가 0보다 1.6미터 위에 있음을 의미합니다.
큐브의 중심은 카메라 앞에서 2미터입니다. 각 큐브
1x1x1미터 크기입니다. 이것은 VR이 *실제 세계* 에 사용자에 반응하여 사물을 조정해야 하기 때문에 중요합니다. 즉 사용자의 움직임과 VR에서의 움직임을 매치 시켜줄 단위가 필요합니다.

그리고 그것으로(단위) 우리는 앞으로 3개의 회전하는 큐브와 VR에 들어갈 버튼 을 카메라 앞에 가져와야 합니다.

{{{example url="../threejs-webvr-basic.html" }}}


카메라 주변에 방(참조할 수 있는 공간)과 같은 감싸는 것이 있으면 VR이 더 잘 작동합니다. 따라서  과거 [the article on backgrounds](threejs-backgrounds.html)에서 진행한 것과 유사하게 간단한 그리드 큐브맵을 추가해 보겠습니다.

우리는 같은 그리드를 사용할 것입니다
그리드 룸으로 제공할 큐브의 각 면에 대한 텍스처입니다.

```js
const scene = new THREE.Scene();
+{
+  const loader = new THREE.CubeTextureLoader();
+  const texture = loader.load([
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+  ]);
+  scene.background = texture;
+}
```

더욱 나아진것을 확인 가능합니다.

{{{example url="../threejs-webvr-basic-w-background.html" }}}

Note: VR을 실제로 보려면 WebXR 호환 장치가 필요합니다.
대부분의 Android 휴대폰은 Chrome 또는 Firefox를 사용하여 WebXR을 지원 합니다.
iOS의 경우 [WebXR App](https://apps.apple.com/us/app/webxr-viewer/id1295998056)를 참조 하세요.
iOS에서 일반적으로 WebXR 지원은 2019년 5월 현재 지원되지 않습니다.


Android 또는 iPhone에서 WebXR을 사용하려면 *VR 헤드셋*이 필요합니다.
전화용. 골판지로 만든 1개에 5달러부터 어디에서나 구입할 수 있습니다.
100달러로. 불행히도 어떤 것을 추천해야 할지 모르겠습니다. 나는 구매했다
그 중 6개는 수년에 걸쳐 생산되었으며 모두 품질이 다릅니다. 나는
약 $25 이상을 지불한 적이 없습니다.

몇 가지 문제만 언급하자면(VR 헤드셋)

1. 휴대전화에 맞습니까?

   전화기는 다양한 크기로 제공되므로 VR 헤드셋이 일치해야 합니다.
   많은 헤드셋이 다양한 크기와 일치한다고 주장합니다. 내 경험
   더 많은 크기가 일치할수록 실제로 더 나빠집니다.
   특정 크기에 맞게 설계되어 타협해야 합니다.
   더 많은 크기를 맞추기 위해. 불행히도 다중 크기 헤드셋이 가장 일반적인 유형입니다.

2. 당신의 얼굴에 집중할 수 있습니까?

   일부 장치에는 다른 장치보다 더 많은 조정이 있습니다. 일반적으로 거기
   최대 2개의 조정입니다. 렌즈가 눈에서 얼마나 멀리 떨어져 있는지
   그리고 렌즈가 얼마나 멀리 떨어져 있는지.

3. 너무 반사적인가요?

   당신의 눈에서 전화까지 플라스틱 원뿔의 많은 헤드셋.
   플라스틱이 반짝이거나 반사되면 다음과 같이 작동합니다.
   화면을 반사하는 거울과 매우 산만합니다.

   리뷰 중 이 문제를 다루는 것으로 보이는 경우는 거의 없습니다.

4. 당신의 얼굴에 편안한가요?

   대부분의 장치는 안경처럼 코에 닿습니다.
   몇 분 후에 아플 수 있습니다. 일부는 주변에 스트랩이 있습니다.
   너의 머리. 다른 사람들은 머리 위로 가는 3번째 끈이 있습니다. 이것들
   장치를 올바른 위치에 유지하는 데 도움이 될 수도 있고 도움이 되지 않을 수도 있습니다.

   대부분의 (모든?) 장치에서 눈이 중앙에 있어야 합니다.
   렌즈와 함께. 렌즈가 자신보다 약간 높거나 낮은 경우
   눈 이미지의 초점이 흐려집니다. 이것은 매우 실망 스러울 수 있습니다.
   일이 초점에서 시작될 수 있지만 45-60초 후에 장치
   1mm 위 또는 아래로 이동했는데 갑자기
   흐릿한 이미지에 초점을 맞추려고 애썼다.

5. 안경을 착용하고 사용할수 있습니까?

   안경을 쓰신 분들은 리뷰를 읽어보시고
   특정 헤드셋이 안경과 잘 매치되는 경우

   정말 아쉽게도 추천을 해드릴 수가 없네요. [구글은 일부
   판지로 만든 저렴한 추천](https://vr.google.com/cardboard/get-cardboard/)
   그들 중 일부는 $ 5만큼 낮으므로 거기에서 시작하고 즐길 수 있습니다.
   그런 다음 업그레이드를 고려하십시오. $5는 커피 1잔 가격과 같으니 꼭 드셔보세요!

또한 3가지 기본 유형의 장치가 있습니다.

1. 3자유도(3dof), 입력 장치 없음

   이것은 일반적으로 전화 스타일이지만 때로는 할 수 있습니다.
   타사 입력 장치를 구입하십시오. 3 자유도
   위/아래(1), 왼쪽/오른쪽(2)을 보고 기울일 수 있음을 의미합니다.
   머리를 좌우로(3).
   
2. 1개의 입력 장치(3dof)로 3자유도(3dof)

   이것은 기본적으로 Google Daydream과 Oculus GO입니다.

   이것들은 또한 3개의 자유도를 허용하고 작은
   VR 내부에서 레이저 포인터처럼 작동하는 컨트롤러입니다.
   레이저 포인터의 자유도는 3개뿐입니다. NS
   시스템은 입력 장치가 가리키는 방향을 알 수 있지만
   장치가 어디에 있는지 알 수 없습니다.

3. 입력 장치(6dof)가 있는 6자유도(6dof)

   이것들은 *좋은 물건*입니다. 6 자유도
   이 장치는 사용자가 보고 있는 방향을 알 뿐만 아니라
   그러나 그들은 또한 당신의 머리가 실제로 어디에 있는지 알고 있습니다. 그 의미는
   왼쪽에서 오른쪽으로 또는 앞뒤로 움직이거나 일어서거나 앉는 경우
   장치는 이것을 등록할 수 있고 VR의 모든 것은 그에 따라 움직입니다.
   으스스하고 놀랍도록 실제적인 느낌입니다. 좋은 데모와 함께
   당신은 날아갈 것입니다. 아니면 적어도 저는 그랬고 지금도 그렇습니다.

   또한 이러한 장치에는 일반적으로 2개의 컨트롤러가 포함됩니다.
   각 손에 대해 그리고 시스템은 귀하의 위치를 정확히 알 수 있습니다.
   손이 어떤 방향으로 향하고 있는지
   손을 뻗고, 만지고,
   밀기, 비틀기 등...


With all that covered I don't for sure know which devices will work with WebXR.
I'm 99% sure that most Android phones will work when running Chrome. You may
need to turn on WebXR support in [`about:flags`](about:flags). I also know Google
Daydream will also work and similarly you need to enable WebXR support in
[`about:flags`](about:flags). Oculus Rift, Vive, and Vive Pro will work via
Chrome or Firefox. I'm less sure about Oculus Go and Oculus Quest as both of
them use custom OSes but according to the internet they both appear to work.

Okay, after that long detour about VR Devices and WebXR
there's some things to cover

* Supporting both VR and Non-VR

  AFAICT, at least as of r112, there is no easy way to support
both VR and non-VR modes with three.js. Ideally
if not in VR mode you'd be able to control the camera using
whatever means you want, for example the `OrbitControls`,
and you'd get some kind of event when switching into and
out of VR mode so that you could turn the controls on/off.

If three.js adds some support to do both I'll try to update
this article. Until then you might need 2 versions of your 
site OR pass in a flag in the URL, something like

```
https://mysite.com/mycooldemo?allowvr=true
```

Then we could add some links in to switch modes

```html
<body>
  <canvas id="c"></canvas>
+  <div class="mode">
+    <a href="?allowvr=true" id="vr">Allow VR</a>
+    <a href="?" id="nonvr">Use Non-VR Mode</a>
+  </div>
</body>
```

and some CSS to position them

```css
body {
    margin: 0;
}
#c {
    width: 100%;
    height: 100%;
    display: block;
}
+.mode {
+  position: absolute;
+  right: 1em;
+  top: 1em;
+}
```

in your code you could use that parameter like this

```js
function main() {
  const canvas = document.querySelector('#c');
  const renderer = new THREE.WebGLRenderer({canvas});
-  renderer.xr.enabled = true;
-  document.body.appendChild(VRButton.createButton(renderer));

  const fov = 75;
  const aspect = 2;  // the canvas default
  const near = 0.1;
  const far = 5;
  const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
  camera.position.set(0, 1.6, 0);

+  const params = (new URL(document.location)).searchParams;
+  const allowvr = params.get('allowvr') === 'true';
+  if (allowvr) {
+    renderer.xr.enabled = true;
+    document.body.appendChild(VRButton.createButton(renderer));
+    document.querySelector('#vr').style.display = 'none';
+  } else {
+    // no VR, add some controls
+    const controls = new OrbitControls(camera, canvas);
+    controls.target.set(0, 1.6, -2);
+    controls.update();
+    document.querySelector('#nonvr').style.display = 'none';
+  }
```

Whether that's good or bad I don't know. I have a feeling the differences
between what's needed for VR and what's needed for non-VR are often
very different so for all but the most simple things maybe 2 separate pages
are better? You'll have to decide.

Note for various reasons this will not work in the live editor
on this site so if you want to check it out 
<a href="../threejs-webvr-basic-vr-optional.html" target="_blank">click here</a>.
It should start in non-VR mode and you can use the mouse or fingers to move
the camera. Clicking "Allow VR" should switch to allow VR mode and you should
be able to click "Enter VR" if you're on a VR device.

* Deciding on the level of VR support

  Above we covered 3 types of VR devices. 

  * 3DOF no input
  * 3DOF + 3DOF input
  * 6DOF + 6DOF input

  You need to decide how much effort you're willing to put in
  to support each type of device.

  For example the simplest device has no input. The best you can
  generally do is make it so there are some buttons or objects in the user's view
  and if the user aligns some marker in the center of the display
  on those objects for 1/2 a second or so then that button is clicked.
  A common UX is to display a small timer that will appear over the object indicating
  if you keep the marker there for a moment the object/button will be selected.

  Since there is no other input that's about the best you can do

  The next level up you have one 3DOF input device. Generally it
  can point at things and the user has at least 2 buttons. The Daydream
  also has a touchpad which provides normal touch inputs.

  In any case if a user has this type of device it's far more
  comfortable for the user to by able to point at things with
  their controller than it is to make them do it with their
  head by looking at things.

  A similar level to that might be 3DOF or 6DOF device with a
  game console controller. You'll have to decide what to do here.
  I suspect the most common thing is the user still has to look
  to point and the controller is just used for buttons.

  The last level is a user with a 6DOF headset and 2 6DOF controllers.
  Those users will find an experience that is only 3DOF to often
  be frustrating. Similarly they usually expect to be able to 
  virtually manipulate things with their hands in VR so you'll
  have to decide if you want to support that or not.

As you can see getting started in VR is pretty easy but
actually making something shippable in VR will require
lots of decision making and design.

This was a pretty brief intro into VR with three.js. We'll
cover some of the input methods in [future articles](threejs-webvr-look-to-select.html).
아니 내가 한거 어디갔냐고 진짜로