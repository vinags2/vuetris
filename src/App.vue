<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

// Game constants
const PLAYFIELD_WIDTH = 10; // Actual playable width (between walls)
const PLAYFIELD_HEIGHT = 20; // Actual playable height
const COLS = 12; // +2 for side walls
const ROWS = 21; // +1 for bottom wall
const BRICK_SIZE = 24; // Size of each brick in the border
const BLOCK_SIZE = 30;
const EMPTY = 0;
const BORDER = 9; // Solid border color (darkest green)

// Tetromino shapes with different colors
const SHAPES = [
  [[1, 1, 1, 1]], // I-piece (line)
  [[1, 1], [1, 1]], // O-piece (square)
  [[1, 1, 1], [0, 1, 0]], // T-piece
  [[1, 1, 1], [1, 0, 0]], // L-piece
  [[1, 1, 1], [0, 0, 1]], // J-piece
  [[1, 1, 0], [0, 1, 1]], // S-piece
  [[0, 1, 1], [1, 1, 0]]  // Z-piece
];

// Color classes for different tetrominoes
const COLORS = [
  'bg-emerald-100', // I-piece (light green)
  'bg-emerald-200', // O-piece
  'bg-emerald-300', // T-piece
  'bg-emerald-400', // L-piece
  'bg-emerald-500', // J-piece
  'bg-emerald-600', // S-piece
  'bg-emerald-700', // Z-piece
  'bg-emerald-800', // Border
  'bg-emerald-900'  // Background
];

// Game state variables
const board = ref(createBoard());
const score = ref(0);
const level = ref(1);
const lines = ref(0);
const gameOver = ref(false);
const isPaused = ref(false);
const nextPiece = ref(null);
const currentPiece = ref(null);
const currentPosition = ref({ x: 0, y: 0 });
const gameLoop = ref(null);

// Computed property for game speed based on level
const speed = computed(() => Math.max(100, 1000 - (level.value - 1) * 100));

/**
 * Initialize a new game
 */
function initGame() {
  board.value = createBoard();
  score.value = 0;
  level.value = 1;
  lines.value = 0;
  gameOver.value = false;
  nextPiece.value = generatePiece();
  spawnPiece();

  // Reset game loop with current speed
  if (gameLoop.value) clearInterval(gameLoop.value);
  gameLoop.value = setInterval(moveDown, speed.value);
}

/**
 * Create an empty board with walls
 * @returns {Array} 2D array representing the game board
 */
function createBoard() {
  // Create a grid with all empty cells
  const grid = Array(ROWS).fill().map(() => Array(COLS).fill(EMPTY));

  // Add solid side walls (will be covered by brick border)
  for (let y = 0; y < ROWS; y++) {
    grid[y][0] = BORDER;
    grid[y][COLS - 1] = BORDER;
  }

  // Add solid bottom wall (will be covered by brick border)
  for (let x = 0; x < COLS; x++) {
    grid[ROWS - 1][x] = BORDER;
  }

  return grid;
}

/**
 * Generate a random tetromino piece
 * @returns {Object} Piece object with shape, color, width, and height
 */
function generatePiece() {
  const shapeIndex = Math.floor(Math.random() * SHAPES.length);
  return {
    shape: SHAPES[shapeIndex],
    color: shapeIndex + 1,
    width: SHAPES[shapeIndex][0].length,
    height: SHAPES[shapeIndex].length
  };
}

/**
 * Spawn a new piece at the top of the playfield
 */
function spawnPiece() {
  currentPiece.value = nextPiece.value || generatePiece();
  nextPiece.value = generatePiece();

  // Spawn pieces centered in the playfield (between walls)
  currentPosition.value = {
    x: Math.floor(COLS / 2) - Math.floor(currentPiece.value.width / 2),
    y: 0
  };

  // Check for game over condition
  if (checkCollision()) {
    gameOver.value = true;
    clearInterval(gameLoop.value);
  }
}

/**
 * Draw the current game state including the falling piece
 * @returns {Array} 2D array representing the current board state
 */
function draw() {
  // Create a new board with the current piece drawn on it
  const newBoard = createBoard();

  // Copy the existing placed blocks (including walls)
  for (let y = 0; y < ROWS; y++) {
    for (let x = 0; x < COLS; x++) {
      // Only copy non-empty cells (preserve the solid borders)
      if (board.value[y][x] !== EMPTY) {
        newBoard[y][x] = board.value[y][x];
      }
    }
  }

  // Draw the current falling piece
  if (currentPiece.value) {
    const { x: pieceX, y: pieceY } = currentPosition.value;
    for (let y = 0; y < currentPiece.value.height; y++) {
      for (let x = 0; x < currentPiece.value.width; x++) {
        if (currentPiece.value.shape[y] && currentPiece.value.shape[y][x]) {
          const boardY = pieceY + y;
          const boardX = pieceX + x;
          // Draw the piece block if within board bounds
          if (boardY >= 0 && boardY < ROWS && boardX >= 0 && boardX < COLS) {
            newBoard[boardY][boardX] = currentPiece.value.color;
          }
        }
      }
    }
  }

  return newBoard;
}

/**
 * Check if the current piece would collide with walls or other pieces
 * @param {number} offsetX - Horizontal offset to check
 * @param {number} offsetY - Vertical offset to check
 * @param {Object} piece - Piece to check (defaults to current piece)
 * @returns {boolean} True if collision would occur
 */
function checkCollision(offsetX = 0, offsetY = 0, piece = currentPiece.value) {
  const { x, y } = currentPosition.value;

  for (let py = 0; py < piece.height; py++) {
    for (let px = 0; px < piece.width; px++) {
      if (piece.shape[py] && piece.shape[py][px]) {
        const newX = x + px + offsetX;
        const newY = y + py + offsetY;

        // Check if piece is outside the play area (including walls)
        if (newX < 0 || newX >= COLS || newY >= ROWS) {
          return true;
        }

        // Check for collision with existing pieces or walls
        if (newY >= 0 && board.value[newY][newX] !== EMPTY) {
          return true;
        }
      }
    }
  }
  return false;
}

/**
 * Merge the current piece into the board permanently
 */
function mergePiece() {
  const { x, y } = currentPosition.value;
  for (let py = 0; py < currentPiece.value.height; py++) {
    for (let px = 0; px < currentPiece.value.width; px++) {
      if (currentPiece.value.shape[py] && currentPiece.value.shape[py][px]) {
        const boardY = y + py;
        const boardX = x + px;
        // Only merge if within bounds (not on walls or below)
        if (boardY >= 0 && boardY < ROWS - 1 && boardX > 0 && boardX < COLS - 1) {
          board.value[boardY][boardX] = currentPiece.value.color;
        }
      }
    }
  }
}

/**
 * Clear completed lines and update score
 */
function clearLines() {
  let linesCleared = 0;

  // Start from the bottom of the play area (above the bottom wall)
  for (let y = ROWS - 2; y >= 0; y--) {
    let isLineComplete = true;

    // Only check the playable area (between walls)
    for (let x = 1; x < COLS - 1; x++) {
      if (board.value[y][x] === EMPTY) {
        isLineComplete = false;
        break;
      }
    }

    if (isLineComplete) {
      // Remove the complete line
      board.value.splice(y, 1);

      // Add a new empty line at the top with borders
      const newRow = Array(COLS).fill(EMPTY);
      newRow[0] = BORDER; // Left border
      newRow[COLS - 1] = BORDER; // Right border
      board.value.unshift(newRow);

      linesCleared++;
      y++; // Check the same row again after shifting
    }
  }

  if (linesCleared > 0) {
    // Update score based on number of lines cleared
    const linePoints = [0, 40, 100, 300, 1200];
    score.value += linePoints[linesCleared] * level.value;
    lines.value += linesCleared;
    level.value = Math.floor(lines.value / 10) + 1;

    // Increase speed for higher levels
    if (gameLoop.value) {
      clearInterval(gameLoop.value);
      gameLoop.value = setInterval(moveDown, speed.value);
    }
  }
}

// Game control functions

/**
 * Move the current piece left if possible
 */
function moveLeft() {
  if (!gameOver.value && !isPaused.value) {
    if (!checkCollision(-1, 0)) {
      currentPosition.value.x--;
    }
  }
}

/**
 * Move the current piece right if possible
 */
function moveRight() {
  if (!gameOver.value && !isPaused.value) {
    if (!checkCollision(1, 0)) {
      currentPosition.value.x++;
    }
  }
}

/**
 * Move the current piece down (called by game loop or player input)
 */
function moveDown() {
  if (gameOver.value || isPaused.value) return;

  if (!checkCollision(0, 1)) {
    currentPosition.value.y++;
  } else {
    // Piece has landed - merge it and spawn next piece
    mergePiece();
    clearLines();
    spawnPiece();
  }
}

/**
 * Rotate the current piece clockwise if possible
 */
function rotate() {
  if (gameOver.value || isPaused.value) return;

  const originalShape = currentPiece.value.shape;

  // Create a new rotated shape (90 degrees clockwise)
  const rotated = [];
  for (let x = 0; x < currentPiece.value.width; x++) {
    const newRow = [];
    for (let y = currentPiece.value.height - 1; y >= 0; y--) {
      newRow.push(originalShape[y][x]);
    }
    rotated.push(newRow);
  }

  const rotatedPiece = {
    ...currentPiece.value,
    shape: rotated,
    width: rotated[0].length,
    height: rotated.length
  };

  // Check if rotation is possible at current position
  if (!checkCollision(0, 0, rotatedPiece)) {
    currentPiece.value = rotatedPiece;
  }
}

/**
 * Drop the current piece instantly to the bottom
 */
function drop() {
  if (gameOver.value || isPaused.value) return;

  // Move down until collision
  while (!checkCollision(0, 1)) {
    currentPosition.value.y++;
  }

  // Merge piece and spawn next
  mergePiece();
  clearLines();
  spawnPiece();
}

/**
 * Toggle game pause state
 */
function togglePause() {
  if (!gameOver.value) {
    isPaused.value = !isPaused.value;
    if (isPaused.value) {
      clearInterval(gameLoop.value);
    } else {
      gameLoop.value = setInterval(moveDown, speed.value);
    }
  }
}

/**
 * Handle keyboard input for game controls
 * @param {KeyboardEvent} e - The keyboard event
 */
function handleKeyDown(e) {
  if (e.key === 'ArrowLeft') moveLeft();
  else if (e.key === 'ArrowRight') moveRight();
  else if (e.key === 'ArrowDown') moveDown();
  else if (e.key === 'ArrowUp') rotate();
  else if (e.key === ' ') { e.preventDefault(); drop(); }
  else if (e.key === 'p' || e.key === 'P') togglePause();
}

// Vue lifecycle hooks
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
  initGame();
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
  if (gameLoop.value) clearInterval(gameLoop.value);
});
</script>

<template>
  <div class="min-h-screen bg-emerald-900 text-emerald-100 flex flex-col items-center justify-center p-4">
    <h1 class="text-4xl font-bold mb-6 text-center font-mono">VueTris</h1>

    <div class="flex flex-col md:flex-row gap-8 items-start">
      <!-- Game Board -->
      <div class="relative bg-emerald-900 p-0 rounded overflow-hidden">
        <!-- Brick Border - Consistent pattern on all sides -->
        <div class="absolute inset-0 pointer-events-none overflow-visible z-20">
          <!-- Left Brick Wall -->
          <div class="absolute left-0 top-0 bottom-0 w-6 bg-emerald-800 shadow-lg">
            <div v-for="i in Math.ceil(ROWS * 2)" :key="'left-' + i"
                 class="h-3 border-b border-emerald-900"
                 :class="{'bg-emerald-700': i % 2 === 0}">
            </div>
          </div>

          <!-- Right Brick Wall -->
          <div class="absolute right-0 top-0 bottom-0 w-6 bg-emerald-800 shadow-lg">
            <div v-for="i in Math.ceil(ROWS * 2)" :key="'right-' + i"
                 class="h-3 border-b border-emerald-900"
                 :class="{'bg-emerald-700': i % 2 === 0}">
            </div>
          </div>

          <!-- Bottom Brick Wall -->
          <div class="absolute left-0 right-0 bottom-0 h-6 bg-emerald-800 shadow-lg">
            <div class="absolute inset-0 flex">
              <div v-for="i in Math.ceil(COLS * 2)" :key="'bottom-' + i"
                   class="h-6 w-3 relative">
                <!-- Brick halves without vertical lines -->
                <div v-if="i % 2 === 0" class="h-3 w-3 bg-emerald-700"></div>
                <div v-else class="h-3 w-3 bg-emerald-800"></div>
                <div v-if="i % 2 === 0" class="h-3 w-3 bg-emerald-800"></div>
                <div v-else class="h-3 w-3 bg-emerald-700"></div>
              </div>
            </div>
          </div>

          <!-- Bottom Left Corner -->
          <div class="absolute left-0 bottom-0 w-6 h-6 bg-emerald-800 shadow-lg overflow-hidden">
            <div class="absolute inset-0">
              <!-- Vertical line -->
              <div class="absolute left-3 top-0 bottom-0 w-px bg-emerald-900"></div>
              <!-- Horizontal line -->
              <div class="absolute left-0 right-0 top-3 h-px bg-emerald-900"></div>
              <!-- Corner pieces -->
              <div class="h-3 w-3 bg-emerald-800 absolute top-0 left-0"></div>
              <div class="h-3 w-3 bg-emerald-700 absolute top-0 right-0"></div>
              <div class="h-3 w-3 bg-emerald-700 absolute bottom-0 left-0"></div>
              <div class="h-3 w-3 bg-emerald-800 absolute bottom-0 right-0"></div>
            </div>
          </div>

          <!-- Bottom Right Corner -->
          <div class="absolute right-0 bottom-0 w-6 h-6 bg-emerald-800 shadow-lg overflow-hidden">
            <div class="absolute inset-0">
              <!-- Vertical line -->
              <div class="absolute left-3 top-0 bottom-0 w-px bg-emerald-900"></div>
              <!-- Horizontal line -->
              <div class="absolute left-0 right-0 top-3 h-px bg-emerald-900"></div>
              <!-- Corner pieces -->
              <div class="h-3 w-3 bg-emerald-800 absolute top-0 left-0"></div>
              <div class="h-3 w-3 bg-emerald-700 absolute top-0 right-0"></div>
              <div class="h-3 w-3 bg-emerald-700 absolute bottom-0 left-0"></div>
              <div class="h-3 w-3 bg-emerald-800 absolute bottom-0 right-0"></div>
            </div>
          </div>
        </div>

        <!-- Game Grid -->
        <div
          v-for="(row, y) in draw()"
          :key="y"
          class="flex"
        >
          <div
            v-for="(cell, x) in row"
            :key="x"
            :class="[
              'w-6 h-6 relative z-0',
              // Only show grid lines for empty cells and not for the border cells
              cell === EMPTY ? 'border border-emerald-800' : 'border border-emerald-900',
              cell !== EMPTY ? COLORS[cell - 1] : 'bg-emerald-900/80',
              cell === BORDER ? '!border-emerald-900' : ''
            ]"
          ></div>
        </div>

        <!-- Game Over Overlay -->
        <div
          v-if="gameOver"
          class="absolute inset-0 bg-black bg-opacity-75 flex flex-col items-center justify-center z-30"
        >
          <h1 class="text-3xl font-bold text-emerald-500 mb-6">Game Over</h1>
          <button
            @click="initGame"
            class="px-6 py-2 bg-emerald-600 hover:bg-emerald-500 rounded font-bold transition-colors"
          >
            Play Again
          </button>
        </div>

        <!-- Pause Overlay -->
        <div
          v-else-if="isPaused"
          class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center z-30"
        >
          <div class="text-3xl font-bold">PAUSED</div>
        </div>
      </div>

      <!-- Game Info Panel -->
      <div class="bg-emerald-800 p-6 rounded-lg w-64">
        <!-- Next Piece Preview -->
        <div class="mb-6">
          <h2 class="text-xl font-bold mb-2">Next Piece:</h2>
          <div class="bg-emerald-900 p-4 rounded flex justify-center items-center h-20">
            <!-- Fixed size container for next piece to prevent layout shifts -->
            <div class="grid grid-cols-4 gap-0.5 w-20 h-16">
              <template v-if="nextPiece">
                <!-- Create a 4x4 grid and position the piece in the center -->
                <div
                  v-for="gridY in 4"
                  :key="'grid-' + gridY"
                  class="contents"
                >
                  <div
                    v-for="gridX in 4"
                    :key="'cell-' + gridX + '-' + gridY"
                    :class="[
                      'w-4 h-4',
                      shouldShowNextPieceBlock(gridX - 1, gridY - 1)
                        ? COLORS[nextPiece.color - 1]
                        : 'bg-transparent'
                    ]"
                  ></div>
                </div>
              </template>
            </div>
          </div>
        </div>

        <!-- Game Statistics -->
        <div class="space-y-4">
          <div>
            <div class="text-emerald-300">Score</div>
            <div class="text-2xl font-bold">{{ score.toLocaleString() }}</div>
          </div>

          <div>
            <div class="text-emerald-300">Level</div>
            <div class="text-2xl font-bold">{{ level }}</div>
          </div>

          <div>
            <div class="text-emerald-300">Lines</div>
            <div class="text-2xl font-bold">{{ lines }}</div>
          </div>

          <!-- Game Controls -->
          <div class="pt-4 space-y-2">
            <button
              @click="togglePause"
              :disabled="gameOver"
              class="w-full py-2 bg-emerald-700 hover:bg-emerald-600 disabled:bg-emerald-800 disabled:opacity-50 rounded font-bold transition-colors"
            >
              {{ isPaused ? 'Resume' : 'Pause' }}
            </button>
            <button
              @click="initGame"
              class="w-full py-2 bg-emerald-600 hover:bg-emerald-500 rounded font-bold transition-colors"
            >
              New Game
            </button>
          </div>

          <!-- Control Instructions -->
          <div class="pt-4 text-sm text-emerald-400">
            <div class="font-bold mb-1">Controls:</div>
            <div>← → : Move left/right</div>
            <div>↑ : Rotate piece</div>
            <div>↓ : Soft drop</div>
            <div>Space : Hard drop</div>
            <div>P : Pause game</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * Helper function to determine if a block should be shown in the next piece preview
 * Centers the piece in a 4x4 grid to prevent layout shifts
 * @param {number} gridX - X position in the 4x4 preview grid (0-3)
 * @param {number} gridY - Y position in the 4x4 preview grid (0-3)
 * @returns {boolean} True if a block should be displayed at this position
 */
function shouldShowNextPieceBlock(gridX, gridY) {
  if (!this.nextPiece) return false;

  const piece = this.nextPiece;

  // Calculate offset to center the piece in the 4x4 grid
  const offsetX = Math.floor((4 - piece.width) / 2);
  const offsetY = Math.floor((4 - piece.height) / 2);

  // Calculate piece coordinates
  const pieceX = gridX - offsetX;
  const pieceY = gridY - offsetY;

  // Check if this grid position corresponds to a piece block
  if (pieceY >= 0 && pieceY < piece.height &&
    pieceX >= 0 && pieceX < piece.width) {
    return piece.shape[pieceY] && piece.shape[pieceY][pieceX];
  }

  return false;
}

export default {
  methods: {
    shouldShowNextPieceBlock
  }
}
</script>