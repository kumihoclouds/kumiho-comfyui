🦊 ComfyUI Custom Node — Quick Project Requirement Documentation (Q-PRD)

Lineage 추적 / 모델 관리 / 그래프 업로드 자동화 플러그인

🔶 1. Overview (개요)

본 프로젝트는 ComfyUI 환경에서 생성된 이미지·워크플로우·모델·LoRA 사용 내역을 자동으로 수집하고,
이를 Kumiho Cloud Graph Lineage 시스템으로 업로드하는 Custom Node 개발을 목표로 한다.

기존 ComfyUI는 PNG 메타데이터에 생성 정보를 포함하지만:

세션 단위 기록

프로젝트 단위 영구 보존 없음

그래프 기반 lineage 업로드 불가

모델/LoRA 레지스트리 없음

→ 따라서 이 Custom Node는 ComfyUI → Kumiho Cloud 간의 “Lineage Bridge” 역할을 수행한다.

🔶 2. Scope (범위)

본 Custom Node는 아래 기능을 제공한다:

✔ (1) Output Lineage 자동 추출

출력 이미지의 PNG metadata 읽기

ComfyUI JSON workflow 추출

다음 정보 파싱:

모델, LoRA, 체크포인트

Sampler, Steps, CFG

시드, 해상도

모든 사용된 노드 리스트

✔ (2) ComfyUI 실행 시점에 자동 이벤트 hook

노드 실행 성공 시

이미지 저장 시

워크플로우 변환 시

→ 이 이벤트를 기준으로 자동 기록.

✔ (3) Kumiho Cloud API로 자동 업로드

업로드 대상:

항목	내용
Artifact	이미지, 워크플로우 JSON, 원본 prompt
Item(kind=model)	체크포인트 모델
Item(kind=lora)	LoRA
Revision	이미지 생성 시점의 기록
Edge	“CREATED_FROM”, “USED_MODEL”, “USED_LORA” 등 lineage edge
✔ (4) ComfyUI-compatible UI 노드 제공

API Key / User Token 입력 필드

Export 모드 (Auto / Manual)

Lineage Preview 패널 (간단 Inspect)

🔶 3. Non-Scope (비범위)

ComfyUI 자체 실행제어(스케줄링/Job 관리)

이미지 업로드(파일 업로드는 Optional; 경로만 저장)

LoRA/모델 파일 자체 저장(메타데이터만 전송)

로컬 파일 정리/삭제 제어

🔶 4. User Workflow (사용 흐름)
(1) ComfyUI에서 워크플로우 실행

사용자는 기존대로 그래프를 실행한다.

(2) 이미지 생성 시 Custom Node 트리거

Custom Node는 다음을 자동 수행:

최종 PNG metadata 읽기

workflow JSON 읽기

사용된 Model/LoRA 식별

(3) Kumiho Cloud로 Lineage 업로드

업로드에서 생성되는 Graph:

GeneratedImage (resource)
  ├─ CREATED_FROM → Workflow(revision)
  ├─ USED_MODEL → Model(item)
  ├─ USED_LORA → Lora(item)
  └─ USED_NODE → Node items(optional)

(4) Web Dashboard에서 lineage 즉시 시각화

사용자는 웹 UI에서 생성 계보를 확인한다.

🔶 5. Functional Requirements (기능 요구사항)
📌 FR-1. Metadata Extractor

PNG 메타데이터 읽기

필드: prompt, negative prompt, seed, cfg, steps, sampler

ComfyUI workflow JSON 추출

모델/LoRA 패스 파싱

📌 FR-2. Model & LoRA Registry Resolver

모델 파일명 → Model Item (Kind=model)

LoRA 파일명 → Item(kind=lora)

이미 존재하면 재사용, 없으면 생성

📌 FR-3. Graph Lineage API 호출

Create Item

Create Revision

Create Artifact

Create Edge(CREATED_FROM, USED_MODEL 등)

📌 FR-4. Auto Mode

이미지 생성 즉시 서버로 전송

워크플로우 수정 시 자동 업데이트

📌 FR-5. Manual Mode

노드를 배치하여 특정 이미지 또는 특정 workflow만 전송

버튼형태 “Upload lineage now”