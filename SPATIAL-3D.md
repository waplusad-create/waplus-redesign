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

## Atlas 결과물 연결 — 실제 검증 결과 (2026-11-20)

- **.spz 스플랫 경로 검증 완료**: Marble 공식 샘플 내보내기 파일
  (`elegant_library_with_fireplace_500k.spz`, World Labs 문서 공개 링크)을
  `SPATIAL_ATLAS.showroom.splatUrl`에 넣어 실제 브라우저에서 렌더 확인.
  - `@sparkjsdev/spark` 2.4.0은 **three 0.180.0 이상**을 요구합니다(importmap 버전 고정 주의).
  - Marble/World Labs 월드는 OpenCV 좌표계(+Y 아래)이므로 `splat.scale.set(1, -1, -1)` 보정이 필요합니다.
  - 로드 후 `splat.getBoundingBox()`로 카메라를 자동 프레이밍합니다(실내 눈높이 기준).
- **Marble 공유 링크는 iframe 임베드 불가**: `marble.worldlabs.ai/world/<id>` 응답에
  `X-Frame-Options: DENY`, `content-security-policy: frame-ancestors 'none'` 이 있습니다.
  그래서 `posterUrl`(대표 이미지) + `externalUrl`(새 창에서 열기) 조합을 사용합니다.

```js
showroom: {
  posterUrl: 'assets/marble-showroom.jpg',            // 월드 뷰어 캡처 이미지
  externalUrl: 'https://marble.worldlabs.ai/world/…', // Copy link 공유 링크
  view: { pos: [0, 1.6, 4], target: [0, 1.4, 0] }      // 카메라 고정(선택)
}
```

## 요금·라이선스 메모 (2026-11 기준, 공식 문서 기준)

| 플랜 | 월 요금 | 월드 생성 | 내보내기 | 상업적 이용 |
| --- | --- | --- | --- | --- |
| Free | $0 | 최대 4개 | 불가(공유 링크만) | 불가 |
| Standard | $20 | 최대 12개 | .spz/.ply 등 | 불가 |
| Pro | $35 | 최대 25개 | .glb 고품질 메시 포함 | **가능** |
| Max | $95 | 최대 75개 | 전체 | 가능 |

- **사이트에 상업적으로 쓰려면 Pro 이상**이 필요합니다(상업적 이용 권리는 Pro부터).
- World API(platform.worldlabs.ai)는 별도 과금: $1 = 1,250 크레딧, 최소 $5.
