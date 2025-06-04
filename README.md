
# WallGoRANI Online

A web-based multiplayer variant of **WallGoRANI**, inspired by *The Devil's Plan*.  
The single-page application (`wallgo.html`) is written in vanilla JavaScript, uses **Tailwind CSS** for styling,
and relies on **Firebase** (Authentication + Firestore) for real‑time sync.

## Features
- Host or join games via a shareable **Game ID**
- 2‑ or 3‑player modes, real‑time chat, and turn timer  
- Anonymous or custom‑token Firebase auth
- Dice roll to decide play order, area‑scoring logic
- Fully responsive; no build step required

## Quick Start

1. **Create a Firebase project** (console.firebase.google.com) and add a **Web App**.  
   Copy the auto‑generated `firebaseConfig` snippet.
2. **Open `wallgo.html`** and find the placeholder:

   ```js
   const firebaseConfig = {
     apiKey: "",          // <‑‑ paste here
     authDomain: "",
     projectId: "",
     storageBucket: "",
     messagingSenderId: "",
     appId: "",
     measurementId: ""
   };
   ```

   > 🛑 **Important:** The page **will not work** until you fill in a **valid `apiKey`** and the remaining
   > Firebase credentials.

3. Enable **Anonymous Authentication** (or your preferred method) and create a **Firestore** database (Native mode).
4. Serve the file from any static HTTP server:

   ```bash
   npx serve .
   # or
   python -m http.server
   ```

## Game Rules / 게임 규칙

### Overview / 개요
**EN** – WallGoRANI is a territory‑control game played on a square grid (size 3–11).  
**KR** – WallGoRANI는 3×3부터 11×11까지의 정사각형 격자에서 영역을 확보하는 보드게임입니다.

### Setup / 준비
| EN | KR |
| --- | --- |
| Host chooses the grid size (3–11). | 호스트가 격자 크기(3–11)를 선택합니다. |
| Each player selects **1–5 pieces** that fit on the grid. | 각 플레이어는 **1–5개의 말**을 선택해 배치합니다. |
| A dice roll decides play order. | 주사위를 굴려 선공을 정합니다. |

### Phases and Turn Structure / 단계와 턴 진행
1. **Piece Placement / 말 배치**  
   - **EN** – Players alternate placing one piece on any empty cell until all pieces are on the board.  
   - **KR** – 플레이어가 번갈아가며 빈 칸에 말을 하나씩 두고, 모든 말을 올릴 때까지 반복합니다.

2. **Main Turn / 메인 턴**  
   Each turn has up to **3 sub‑steps**:  
   | Step | EN | KR |
   | --- | --- | --- |
   | ① Select Piece | Choose one of your own pieces. | 자신의 말을 하나 선택합니다. |
   | ② Move (≤2) | Move it **orthogonally one cell** (up/down/left/right). You may move the **same piece up to 2 times** per turn. | 선택한 말을 **상하좌우로 한 칸씩**, 최대 **2번** 이동할 수 있습니다. |
   | ③ Place Wall | After moving (or standing still), place **one wall segment** (horizontal or vertical) adjacent to that piece. A wall cannot overlap another wall or extend outside the board. | 이동을 마친 후(또는 이동을 생략한 후) 해당 말과 인접한 칸 경계에 **벽 세그먼트 1개**를 설치합니다. 기존 벽과 겹치거나 보드를 벗어날 수 없습니다. |

   > If you skip movement by clicking the same piece, you still proceed to wall placement.  
   > 말을 선택한 뒤 같은 자리를 다시 클릭하면 이동을 생략하고 곧바로 벽 설치 단계로 넘어갑니다.

3. **Timeout & Penalty / 시간 제한 & 패널티**  
   - **EN** – Failing to place a wall in time triggers an automatic random wall placement as a penalty.  
   - **KR** – 제한 시간 내에 벽을 설치하지 못하면, 패널티로 무작위 위치에 벽이 자동 설치됩니다.

### Scoring / 점수 계산
**EN** – Fully walled‑off regions are automatically detected after each turn.  
- If a region contains *only one player’s* pieces, that player earns points equal to the number of enclosed cells.  
- Regions containing pieces from multiple players are **contested** and yield no points.  

**KR** – 매 턴 종료 시 완전히 둘러싸인 영역을 자동으로 탐지합니다.  
- **단일 플레이어**의 말만 포함된 영역은 해당 플레이어가 그 영역의 칸 수만큼 점수를 얻습니다.  
- 여러 플레이어의 말이 섞여 있으면 **분쟁 지역**으로 간주되어 점수가 없습니다.

### End of Game / 게임 종료
| EN | KR |
| --- | --- |
| The game ends when **all regions are non‑contested** (i.e., every cell belongs to exactly one player) or no legal moves remain. | **모든 영역이 분쟁 지역이 아니게 되거나** 더 이상 합법적인 이동이 없으면 게임이 종료됩니다. |
| The player with the highest cumulative area score wins. | 최종 점수(획득 영역 칸 수)가 가장 높은 플레이어가 승리합니다. |

## Data Layout (Firestore)

```
artifacts/
  └─ {appId}/
     └─ public/
        └─ data/
           └─ wall_baduk_games/
              └─ {gameId}/            # Individual game document
                 ├─ chatMessages/     # Sub‑collection for chat
                 └─ …                # Game state fields
```

## Security Notes
- **No default security rules are provided**. Restrict read/write access to authenticated users, e.g.:
  ```
  match /artifacts/{appId}/public/data/wall_baduk_games/{gameId} {
    allow read, write: if request.auth != null;
  }
  ```
- Consider limiting document sizes and write rates for cost control.

## File List
| Path          | Description                     |
|---------------|---------------------------------|
| `wallgo.html` | Main SPA                        |
| `README.md`   | Setup, rules & usage guide      |

## License
MIT
