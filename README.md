# FLUX 2 Pro API (`flux-2-pro`)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

<p align="center">
  <img src="assets/01-product-hero-perfume.jpg" width="820" alt="Sample render via the APIMart image API">
</p>

> **from $0.024 per image** (1MP $0.024 · 2MP $0.036 · 3MP $0.048) — `https://api.apimart.ai/v1` OpenAI 호환 엔드포인트 하나면 되고, 월 구독 플랜은 필요 없습니다. *(2026-10-12 기준)*

**[API 키](https://go.apimart.ai/k-3f235f)** · **[가격](https://go.apimart.ai/k-e461ef)** · **[모델 페이지](https://go.apimart.ai/k-e17a13)** · [⚡ 60초 퀵스타트](#quickstart)

**팀들이 APIMart를 통해 FLUX 2 Pro(`flux-2-pro`)을 쓰는 이유**

- **키 하나로 카탈로그 전체를.** 동일한 `https://api.apimart.ai/v1` base URL과 `Authorization` 헤더로 FLUX 2 Pro(`flux-2-pro`)과 이미지·비디오·언어 모델 300+ 개에 접근할 수 있습니다. 바꾸는 건 `model` 필드뿐이고 클라이언트는 그대로입니다.
- **최소 $1, 종량제.** 구독도, 사전에 규모를 정해야 하는 선불 플랜도 없습니다. $1부터 충전해 호출한 만큼만 쓰면 됩니다.
- **과금 내역은 응답으로 돌아옵니다.** 완료된 모든 작업은 청구 금액(`cost` / `credits_cost`)을 보고하고, `failed`로 표시된 작업은 환불됩니다.
- **비동기 설계.** 제출하고 `task_id`를 받아 `GET /v1/tasks/{id}`를 폴링하면 끝입니다. 배치와 재시도는 별도 통합이 아니라 평범한 큐 작업입니다.

<!-- /conv-kit:v1 -->

**다른 이름:** 플럭스

FLUX 2 Pro by Black Forest Labs generates images from text or references, priced per megapixel by output size.

## Model id and routes

| Route | `model` value | Billing | Notes |
| --- | --- | --- | --- |
| Per-image | `flux-2-pro` | per delivered image, by resolution | see the [model page](https://go.apimart.ai/k-e17a13) for the exact tiers |

Endpoint: `POST https://api.apimart.ai/v1/images/generations`, then poll `GET /v1/tasks/{task_id}`.

## Pricing

<!-- pricing:model:start -->
| Output | List price | Effective price |
| --- | --- | --- |
| 1MP | 0.03 | 0.024 |
| 2MP | 0.045 | 0.036 |
| 3MP | 0.06 | 0.048 |
| 4MP | 0.075 | 0.06 |


<!-- pricing:model:end -->

Prices are a snapshot; the [pricing page](https://go.apimart.ai/k-e461ef) and [`data/model.json`](data/model.json) are refreshed by CI, and a completed task reports the exact amount in its `cost` field.

## Request parameters

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `model` | string | required | the model id shown in the route table above |
| `prompt` | string | required | what to generate, or what to change when editing |
| `size` | string | `auto` / `1:1` | aspect ratio, or `auto` where the route supports it |
| `resolution` | string | `1K` | output resolution tier — cost scales with it |
| `n` | integer | `1` | number of images per task, where the route allows more than one |
| `image_urls` | string[] | — | reference images as public HTTP(S) URLs or base64 data URLs |
| `nsfw_check` | boolean | `false` | runs `omni-moderation-latest` before submitting |

The table lists the fields accepted by the APIMart `images/generations` endpoint. Check the [model page](https://go.apimart.ai/k-e17a13) for anything this model adds on top.

## Quickstart

```bash
curl --request POST --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  -d '{"model":"flux-2-pro","prompt":"A bamboo forest at sunrise, slow push-in","size":"1:1","resolution":"1K"}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

created = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
    "model": "flux-2-pro", "prompt": "A bamboo forest at sunrise, slow push-in",
}).json()
task_id = created["data"]["id"]

while True:
    task = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(5)
print(task.get("cost"), task["result"]["images"][0]["url"])
```

Runnable versions: [`examples/`](examples). The task lifecycle is `pending → processing → completed | failed`.

## FAQ

**FLUX 2 Pro 모델 id는 무엇인가요?**

`flux-2-pro`입니다. 위의 라우트 표를 참고하세요.

**과금 방식은 어떻게 되나요?**

**from $0.024 per image** (1MP $0.024 · 2MP $0.036 · 3MP $0.048). 완료된 작업은 `cost` 필드에 정확한 금액을 보고하고, 실패한 작업은 환불됩니다.

**How do I get a key?**

Use the [Get an API key](https://go.apimart.ai/k-3f235f) link — $1 minimum top-up, no plan required.

<!-- related-repos:auto -->
## Related repositories

- **Same model, this language** — the other half of this pair: [flux-2-prompts-ko](https://github.com/onnxrnf/flux-2-prompts-ko)
- **Same model, other languages** — 9 more language builds: 中文 [flux-2-api-cn](https://github.com/zx85rjc583qz/flux-2-api-cn) · Deutsch [flux-2-api-de](https://github.com/kx4u3/flux-2-api-de) · Español [flux-2-api-es](https://github.com/xx54rb3/flux-2-api-es) · Français [flux-2-api-fr](https://github.com/pv7hg/flux-2-api-fr) · Relay/gateway `flux-2-api-gateway` · Bahasa [flux-2-api-id](https://github.com/l56di0sfnbot/flux-2-api-id) · 日本語 [flux-2-api-ja](https://github.com/c5yg2uk/flux-2-api-ja) · Português [flux-2-api-pt](https://github.com/dtqipdogsna0/flux-2-api-pt) · Русский [flux-2-api-ru](https://github.com/ndn2ib6wgfy/flux-2-api-ru)
- **More models in this language** — [midjourney-api-ko](https://github.com/ocvewn2-9ueo/midjourney-api-ko) · [gemini-2-5-flash-image-preview-api-ko](https://github.com/uiyflzgo/gemini-2-5-flash-image-preview-api-ko) · [nano-banana-api-ko](https://github.com/l-2jb09sq/nano-banana-api-ko) · [gemini-3-1-flash-image-preview-api-ko](https://github.com/sgc3a85h2-33l/gemini-3-1-flash-image-preview-api-ko) · [nano-banana-2-api-ko](https://github.com/jjb18r/nano-banana-2-api-ko) · [gemini-3-pro-image-preview-api-ko](https://github.com/a-me4/gemini-3-pro-image-preview-api-ko)
- **Topic hubs** — every build of this model, all languages: [flux-2](https://github.com/topics/flux-2) · [flux-2-api](https://github.com/topics/flux-2-api)

This repository is one build in a cross-linked set of API guides for the same model family; each language build links back to the others.
<!-- /related-repos:auto -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Model page | <https://go.apimart.ai/k-e17a13> | `apimart.ai/ko/model/flux-2` |
| Live pricing | <https://go.apimart.ai/k-e461ef> | `apimart.ai/ko/pricing` |
| Get an API key | <https://go.apimart.ai/k-3f235f> | `apimart.ai/ko/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

FLUX 2 Pro is a third-party model served through APIMart; this repository documents how to call it and publishes
model ids and prices, and does not claim official status. Model names, prices and documentation belong to their
respective owners.

## License

MIT — see [LICENSE](LICENSE).
