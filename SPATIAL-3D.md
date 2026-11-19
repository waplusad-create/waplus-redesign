# 3D 공간 섹션 (World Labs Atlas 연동)

`index.html`의 `#spatial` 섹션은 three.js 기반 인터랙티브 3D 뷰어입니다.
별도 빌드 없이 `index.html` 한 파일로 동작하며, three.js는 jsDelivr CDN에서
importmap으로 불러옵니다(three 0.180.0).

## 지금 상태

- 드래그 360° 회전 / 휠·핀치 줌 / 자동 회전 토글 / 좌우·확대·축소 버튼
- `K-뷰티 쇼룸`, `AI 퍼포먼스 오피스` 두 공간을 WebGL로 실시간 렌더링
- 공간 전환 시 카메라 트윈 + 캡션 크로스페이드
- 섹션이 화면 밖이면 렌더 루프를 멈춰 성능을 아낌
- WebGL을 못 쓰는 환경에서는 안내 문구로 대체(`.spatial-fallback`)

## Atlas 결과물 연결 방법

`index.html`의 `SPATIAL_ATLAS` 객체에 URL만 넣으면 됩니다.

```js
const SPATIAL_ATLAS = {
  showroom: {
    splatUrl: 'https://.../showroom.spz',  // Atlas Gaussian Splat (.spz / .ply) → Spark 렌더러
    glbUrl: null,                          // Atlas Splats-to-Mesh 결과 (.glb) → GLTFLoader
    embedUrl: null,                        // Atlas 공유/임베드 URL → iframe으로 표시
    posterUrl: null,                       // 카메라 워킹 mp4 또는 대표 이미지
    view: null                             // { pos:[x,y,z], target:[x,y,z] } 카메라 고정(선택)
  },
  office: { splatUrl: null, glbUrl: null, embedUrl: null, posterUrl: null }
};
```

- `.spz`를 넣으면 `@sparkjsdev/spark`(CDN 2.4.0, three 0.180 이상 필요)를 그때만 동적으로 불러와 씬에 추가하고,
  절차적으로 만든 공간은 숨깁니다. 콘솔에서 `WAPLUS_SPATIAL.applyAtlas('showroom')`로 즉시 재적용할 수 있습니다.
- `.glb`는 원점·크기를 자동 정규화해 바닥에 맞춰 배치합니다.
- `embedUrl`을 넣으면 캔버스 대신 iframe이 표시됩니다.
- URL이 비어 있으면 지금의 WebGL 공간이 그대로 보입니다.

## 배포

- 카페24(`waplez.cafe24.com`)에는 `index.html`과 `hero_bg_final.mp4`만 올리면 됩니다.
  three.js는 CDN이라 추가 업로드가 필요 없습니다.
- `three@0.180.0` CDN 경로는 `index.html` 상단 importmap에서 관리합니다.
