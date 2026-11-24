# Basys-3 Digital Watch & Stopwatch

Basys-3 FPGA 보드에서 디지털 시계와 스톱워치를 설계하고, 4자리 FND로 통합 제어하는 시스템입니다.

- **기간**: 2026.10 (대한상공회의소 AI 시스템반도체 설계 2기 미니 프로젝트)
- **인원**: 2명 (정준혁, 김성은)
- **툴/보드**: Verilog HDL, Vivado, Digilent Basys-3 (Artix-7 XC7A35T)

## 주요 기능

- 100 MHz 클럭에서 100 Hz tick을 만들어 Watch(현재 시각)와 Stopwatch(경과 시간)가 **독립적으로 동작**하고, `sw[0]`로 출력만 선택
- 버튼 디바운스(100 Hz 샘플링 8회 연속 HIGH, 약 80 ms) + FSM 제어
- `sw[1]`로 표시 형식 전환 (hour.min / sec.msec)

| 입력 | Watch (`sw[0]=0`) | Stopwatch (`sw[0]=1`) |
|---|---|---|
| btnC | reset → 12:00:00.00 | reset → 00:00:00.00 |
| btnL | 설정 자리 ← 이동 | CLEAR (STOP 상태에서만) |
| btnR | 설정 자리 → 이동 | RUN / STOP 전환 |
| btnU | 선택 자리 +1 / +10 | UP ↔ DOWN 모드 (STOP 상태에서만) |
| btnD | 선택 자리 −1 / −10 | - |

## 구조

- **Stopwatch Control Unit**: 2-bit 4-state FSM (STOP · RUN · CLEAR · MODE)
- **Watch Control Unit**: 2-bit 4-state FSM, 상태 = 설정 중인 FND 자리 (S1 · S10 · S100 · S1000)
- 범위: msec 0~99, sec · min 0~59, hour 0~23

## 검증

테스트 케이스별로 입력 → 관찰 신호 → 기대 결과를 정의하고 파형으로 확인했습니다.
Stopwatch RUN/STOP · UP/DOWN 전환, Watch 자리 이동과 증감, 모드 전환 MUX를 검증했습니다.

**발견한 버그**: Watch 시 설정에서 14시에 +10을 누르면 24로 표시
- 원인: +10 연산 후 범위 보정이 없음
- 수정 후: 13 → 23 (+10 유지), 14 → 04 (보정), 24 이상 미출력

**남은 검증 과제**: sec → min → hour 전체 자리 올림, 최댓값 순환, 수동 설정과 자동 tick이 동시에 일어나는 조건

## 파일

```
rtl/          watch.v (top), watch_control_unit.v, watch_datapath.v,
              stopwatch_cu.v, stopwatchdatapath.v, fnd_controller.v, btn_db.v
constraints/  basys3.xdc
docs/         report.pptx (결과 보고서)
```
