소개
이제 조명이 있으므로 그림자가 필요합니다. 물체의 뒷면은 실제로 어둠 속에 있으며 이것을 코어 섀도우 라고 합니다 . 우리가 놓치고 있는 것은 drop shadow 입니다 . 여기서 객체는 다른 객체에 그림자를 만듭니다.

그림자는 항상 실시간 3D 렌더링의 도전 과제였으며 개발자는 합리적인 프레임 속도로 사실적인 그림자를 표시하는 트릭을 찾아야 합니다.

이를 구현하는 방법에는 여러 가지가 있으며 Three.js에는 솔루션이 내장되어 있습니다. 이 솔루션은 편리하지만 완벽하지는 않습니다.

작동 방식
그림자가 내부적으로 어떻게 작동하는지 자세히 설명하지는 않겠지만 기본 사항을 이해하려고 노력할 것입니다.

한 번의 렌더링을 수행할 때 Three.js는 먼저 그림자를 드리우는 각 조명에 대해 렌더링을 수행합니다. 이러한 렌더는 조명이 카메라인 것처럼 보이는 것을 시뮬레이션합니다. 이러한 조명이 렌더링되는 동안 MeshDepthMaterial 은 모든 메시 재질을 대체합니다.

결과는 텍스처와 명명된 섀도우 맵으로 저장됩니다.

이러한 그림자 맵을 직접 볼 수는 없지만 그림자를 수신하고 형상에 투영되는 모든 재료에 사용됩니다.

다음은 방향 조명과 스포트라이트가 보는 것에 대한 훌륭한 예입니다. https://threejs.org/examples/webgl_shadowmap_viewer.html

설정
우리의 스타터는 하나의 방향 조명과 하나의 주변 조명이 있는 평면에 하나의 단순한 구로 구성됩니다.

Dat.GUI에서 이러한 조명과 재료의 금속성 및 거칠기를 제어할 수 있습니다.

/assets/lessons/15/step-01.png

그림자를 활성화하는 방법
먼저 다음에서 그림자 맵을 활성화해야 합니다 renderer.

renderer.shadowMap.enabled = true
자바스크립트
그런 다음 장면의 각 개체를 살펴보고 개체가 castShadow속성 으로 그림자를 투사할 수 있는지, 개체가 receiveShadow속성으로 그림자를 받을 수 있는지 결정해야 합니다 .

가능한 한 적은 수의 개체에서 이러한 기능을 활성화하십시오.

sphere.castShadow = true

// ...
plane.receiveShadow = true
자바스크립트
마지막으로 castShadow속성 을 사용하여 조명의 그림자를 활성화합니다 .

다음 유형의 조명만 그림자를 지원합니다.

포인트라이트
디렉셔널 라이트
스포트라이트
그리고 다시 가능한 한 적은 수의 조명에서 그림자를 활성화하십시오.

directionalLight.castShadow = true
자바스크립트
/assets/lessons/15/step-02.png

평면에서 구의 그림자를 얻어야 합니다.

슬프게도, 그 그림자는 끔찍해 보입니다. 개선해 보도록 하겠습니다.

그림자 맵 최적화
렌더링 크기
강의 시작 부분에서 말했듯이 Three.js는 각 조명에 대해 그림자 맵이라는 렌더링을 수행합니다. shadow라이트 의 속성을 사용하여 이 그림자 맵(및 기타 여러 항목)에 액세스할 수 있습니다 .

console.log(directionalLight.shadow)
자바스크립트
렌더링의 경우 크기를 지정해야 합니다. 기본적으로 섀도우 맵 크기는 512x512성능상의 이유로 만 사용됩니다. 개선할 수 있지만 밉매핑에는 2의 거듭제곱 값이 필요합니다.

directionalLight.shadow.mapSize.width = 1024
directionalLight.shadow.mapSize.height = 1024
자바스크립트
/assets/lessons/15/step-03.png

그림자는 이미 더 좋아 보일 것입니다.

가깝고도 멀다
Three.js는 카메라를 사용하여 그림자 맵 렌더링을 수행합니다. 이러한 카메라는 우리가 이미 사용한 카메라와 동일한 속성을 가지고 있습니다. 이는 near및 를 정의해야 함을 의미합니다 far. 그림자의 품질이 향상되지는 않지만 그림자가 보이지 않거나 그림자가 갑자기 잘리는 버그를 수정할 수 있습니다.

우리가 카메라를 디버그하는 데 도움과를 미리 보려면 near하고 far우리가 사용할 수 CameraHelper을 에있는 그림자 맵에 사용되는 카메라 directionalLight.shadow.camera특성 :

const directionalLightCameraHelper = new THREE.CameraHelper(directionalLight.shadow.camera)
scene.add(directionalLightCameraHelper)
자바스크립트
/assets/lessons/15/step-04.png

지금 당신은 시각적으로 볼 수 near및 far카메라. 장면에 맞는 값을 찾으십시오.

directionalLight.shadow.camera.near = 1
directionalLight.shadow.camera.far = 6
자바스크립트
/assets/lessons/15/step-05.png

진폭
방금 추가한 카메라 도우미를 사용하면 카메라의 진폭이 너무 큰 것을 알 수 있습니다.

우리는 DirectionalLight 를 사용하고 있기 때문에 Three.js 는 OrthographicCamera 를 사용하고 있습니다. 당신이 카메라 수업에서 기억한다면, 우리는 카메라가 함께 볼 수있는 방법까지 각 측에 제어 할 수 있습니다 top, right, bottom, 및 left특성. 이러한 속성을 줄여 보겠습니다.

directionalLight.shadow.camera.top = 2
directionalLight.shadow.camera.right = 2
directionalLight.shadow.camera.bottom = - 2
directionalLight.shadow.camera.left = - 2
자바스크립트
/assets/lessons/15/step-06.png

값이 작을수록 그림자가 더 정확해집니다. 그러나 너무 작으면 그림자가 잘립니다.

완료되면 카메라 도우미를 숨길 수 있습니다.

directionalLightCameraHelper.visible = false
자바스크립트
/assets/lessons/15/step-07.png

흐림
radius속성을 사용 하여 그림자 흐림을 제어할 수 있습니다 .

directionalLight.shadow.radius = 10
자바스크립트
/assets/lessons/15/step-08.png

이 기술은 물체와 카메라의 근접성을 사용하지 않습니다. 그것은 단지 일반적이고 값싼 흐림 효과입니다.

그림자 맵 알고리즘
다양한 유형의 알고리즘을 그림자 맵에 적용할 수 있습니다.

THREE.BasicShadowMap 성능은 우수하지만 품질이 좋지 않습니다.
THREE.PCFShadowMap 성능은 떨어지지만 가장자리는 더 부드럽습니다.
THREE.PCFSoftShadowMap 성능은 떨어지지만 가장자리는 더 부드럽습니다.
THREE.VSMShadowMap 성능이 낮고 제약이 많으면 예기치 않은 결과가 발생할 수 있습니다.
변경하려면 renderer.shadowMap.type속성을 업데이트하세요 . 기본값은 THREE.PCFShadowMap이지만 THREE.PCFSoftShadowMap더 나은 품질을 위해 사용할 수 있습니다 .

renderer.shadowMap.type = THREE.PCFSoftShadowMap
자바스크립트
/assets/lessons/15/step-09.png

반경 속성은 에서 작동하지 않습니다 THREE.PCFSoftShadowMap. 선택해야 합니다.

스포트라이트
Lights 단원 에서 했던 것처럼 SpotLight를 추가하고 속성을 에 추가해 보겠습니다 . 에 속성을 추가하는 것을 잊지 마십시오 .castShadowtruetargetscene

카메라 도우미도 추가합니다.

// Spot light
const spotLight = new THREE.SpotLight(0xffffff, 0.4, 10, Math.PI \* 0.3)

spotLight.castShadow = true

spotLight.position.set(0, 2, 2)
scene.add(spotLight)
scene.add(spotLight.target)

const spotLightCameraHelper = new THREE.CameraHelper(spotLight.shadow.camera)
scene.add(spotLightCameraHelper)
자바스크립트
장면이 너무 밝으면 다른 조명 강도를 줄일 수 있습니다.

const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)

// ...

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4)
자바스크립트
/assets/lessons/15/step-10.png

보시다시피 그림자가 잘 병합되지 않습니다. 그것들은 독립적으로 처리되며 불행히도 그것에 대해 할 일이별로 없습니다.

그러나 방향광에 사용한 것과 동일한 기술을 사용하여 그림자 품질을 개선할 수 있습니다.

변경 shadow.mapSize:

spotLight.shadow.mapSize.width = 1024
spotLight.shadow.mapSize.height = 1024
자바스크립트
/assets/lessons/15/step-11.png

이제 내부적 으로 SpotLight 를 사용하고 있기 때문에 Three.js 는 PerspectiveCamera 를 사용하고 있습니다. 즉 top, right, bottom, 및 left속성 대신 fov속성을 변경해야 합니다 . 그림자를 자르지 않고 가능한 한 작은 각도를 찾으십시오.

spotLight.shadow.camera.fov = 30
자바스크립트
/assets/lessons/15/step-12.png

near및 far값을 변경 합니다.

spotLight.shadow.camera.near = 1
spotLight.shadow.camera.far = 6
자바스크립트
/assets/lessons/15/step-13.png

완료되면 카메라 도우미를 숨길 수 있습니다.

spotLightCameraHelper.visible = false
자바스크립트
/assets/lessons/15/step-14.png

포인트라이트
그림자를 지원하는 마지막 조명인 PointLight를 사용해 보겠습니다 .

// Point light
const pointLight = new THREE.PointLight(0xffffff, 0.3)

pointLight.castShadow = true

pointLight.position.set(- 1, 1, 0)
scene.add(pointLight)

const pointLightCameraHelper = new THREE.CameraHelper(pointLight.shadow.camera)
scene.add(pointLightCameraHelper)
자바스크립트
장면이 너무 밝으면 다른 조명 강도를 줄일 수 있습니다.

const ambientLight = new THREE.AmbientLight(0xffffff, 0.3)

// ...

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.3)

// ...

const spotLight = new THREE.SpotLight(0xffffff, 0.3, 10, Math.PI \* 0.3)
자바스크립트
/assets/lessons/15/step-15.png

보시다시피 카메라 도우미는 PerspectiveCamera ( SpotLight 와 같은 )이지만 아래쪽을 향하고 있습니다. Three.js가 PointLight에 대한 그림자 맵을 처리하는 방식 때문 입니다 .

포인트 라이트는 모든 방향으로 비추기 때문에 Three.js는 큐브 섀도우 맵을 생성하기 위해 6개의 방향을 각각 렌더링해야 합니다. 카메라 도우미는 6개의 렌더링 중 마지막(아래쪽)에서 카메라의 위치입니다.

이러한 모든 렌더링을 수행하면 성능 문제가 발생할 수 있습니다. 그림자가 활성화된 상태에서 너무 많은 PointLight 를 사용 하지 않도록 하십시오 .

여기서 조정할 수 있는 유일한 속성은 mapSize, near및 입니다 far.

pointLight.shadow.mapSize.width = 1024
pointLight.shadow.mapSize.height = 1024

pointLight.shadow.camera.near = 0.1
pointLight.shadow.camera.far = 5
자바스크립트
/assets/lessons/15/step-16.png

완료되면 카메라 도우미를 숨길 수 있습니다.

pointLightCameraHelper.visible = false
자바스크립트
/assets/lessons/15/step-17.png

베이킹 섀도우
Three.js 그림자는 장면이 단순하면 매우 유용할 수 있지만 그렇지 않으면 지저분해질 수 있습니다.

좋은 대안은 구운 그림자입니다. 우리는 이전 수업에서 구운 조명에 대해 이야기했고 정확히 같은 것입니다. 그림자는 재료에 적용하는 텍스처에 통합됩니다.

모든 그림자 관련 코드 줄을 주석 처리하는 대신 렌더러에서 간단히 비활성화할 수 있습니다.

renderer.shadowMap.enabled = false
자바스크립트
/assets/lessons/15/step-18.png

이제 /static/textures/backedShadow.jpg클래식 TextureLoader 를 사용하여 에 있는 그림자 텍스처를 로드할 수 있습니다 .

/assets/lessons/15/bakedShadow.jpg

개체와 조명을 만들기 전에 다음 코드를 추가합니다.

/\*\*

- Textures
  \*/
  const textureLoader = new THREE.TextureLoader()
  const bakedShadow = textureLoader.load('/textures/bakedShadow.jpg')
  자바스크립트
  마지막으로 평면에서 MeshStandardMaterial 을 사용하는 대신 다음 과 bakedShadow같이 간단한 MeshBasicMaterial을 사용합니다 map.

const plane = new THREE.Mesh(
new THREE.PlaneGeometry(5, 5),
new THREE.MeshBasicMaterial({
map: bakedShadow
})
)
자바스크립트
/assets/lessons/15/step-19.png

흐릿하고 사실적인 가짜 그림자가 보일 것입니다. 주요 문제는 동적이지 않고 구 또는 조명이 움직이면 그림자가 움직이지 않는다는 것입니다.

/assets/lessons/15/step-20.png

베이킹 섀도우 대안
덜 현실적이지만 더 역동적인 솔루션은 구 아래와 평면 약간 위에 더 단순한 그림자를 사용하는 것입니다.

/assets/lessons/15/simpleShadow.jpg

질감은 단순한 후광입니다. 흰색 부분은 보이고 검은색 부분은 보이지 않습니다.

그런 다음 구와 함께 그림자를 이동합니다.

먼저 MeshStandardMaterial 을 평면에 다시 배치하여 이전에 구운 그림자를 제거하겠습니다 .

const plane = new THREE.Mesh(
new THREE.PlaneGeometry(5, 5),
material
)
자바스크립트
그런 다음 에 있는 기본 그림자 텍스처를 로드할 수 있습니다 /static/textures/backedShadow.jpg.

const simpleShadow = textureLoader.load('/textures/simpleShadow.jpg')
자바스크립트
회전하고 바닥보다 약간 위에 배치하는 간단한 평면을 사용하여 그림자를 만들 수 있습니다. 재질은 검정색이어야 하지만 그림자 텍스처가 alphaMap. 변경하는 것을 잊지 마세요 transparent에 true, 그리고에 메쉬를 추가합니다 scene:

const sphereShadow = new THREE.Mesh(
new THREE.PlaneGeometry(1.5, 1.5),
new THREE.MeshBasicMaterial({
color: 0x000000,
transparent: true,
alphaMap: simpleShadow
})
)
sphereShadow.rotation.x = - Math.PI \* 0.5
sphereShadow.position.y = plane.position.y + 0.01

scene.add(sphere, sphereShadow, plane)
자바스크립트
/assets/lessons/15/step-21.png

너무 현실적이지는 않지만 매우 성능이 뛰어난 그림자입니다.

구에 애니메이션을 적용하려는 경우 그에 따라 그림자를 애니메이션하고 구의 고도에 따라 불투명도를 변경할 수 있습니다.

const clock = new THREE.Clock()

const tick = () =>
{
const elapsedTime = clock.getElapsedTime()

    // Update the sphere
    sphere.position.x = Math.cos(elapsedTime) * 1.5
    sphere.position.z = Math.sin(elapsedTime) * 1.5
    sphere.position.y = Math.abs(Math.sin(elapsedTime * 3))

    // Update the shadow
    sphereShadow.position.x = sphere.position.x
    sphereShadow.position.z = sphere.position.z
    sphereShadow.material.opacity = (1 - sphere.position.y) * 0.3

    // ...

}

tick()
자바스크립트
어떤 기술을 사용할 것인가
그림자를 처리하는 올바른 솔루션을 찾는 것은 귀하에게 달려 있습니다. 그것은 당신이 알고 있는 프로젝트, 공연 및 기술에 달려 있습니다. 당신은 또한 그들을 결합할 수 있습니다.
