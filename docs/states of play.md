# States of Play

1. On startup (State = IDLE)

   - don't show Pause/Resume
   - don't start game
   - show gameboard (and hide Game Paused Screen)
   - show New Game
   - don't show Next Piece
   - reset Score, Level and Lines
   - gameidle = true

2. On start of game (by clicking New Game) (State = PLAYING)

   - show Pause
   - start game (and hide Game Paused Screen)
   - show gameboard
   - show Next Piece
   - show End Game (with verifiction window if clicked)
   - gameIdle = false, isPaused = false, gameOver = false

3. On Pause (State = PAUSED)

   - show Resume
   - show Game Paused Screen
   - show End Game (with verifiction window if clicked)
   - show Next Piece
   - gameIdle = false, gameOver = false, isPaused = true

4. On Resume (State = RESUMED)

   - show Pause
   - show gameboard
   - show Next Piece
   - show End Game (with verifiction window if clicked)
   - change State to PLAYING
   - gameIdle = false, gameOver = false, isPaused = false

5. On End Game (State = ENDED)
   - verify with user that the game is to end
   - return to On Startup State
   - change State to IDLE

# States of Controls in the different states

1. Pause/Resume button

   - hide when State = IDLE, ENDED
   - show PAUSE when State = PLAYING, RESUMED
   - show RESUME when State = PAUSED

   or

   - hide when gameIdle = true
   - show PAUSE when gameIdle = false, isPaused = false
   - show RESUME when gameIdle = false, isPaused = true

2. Gameboard

   - hide when State = PAUSED
   - show in all other States

   or

   - hide when isPaused = true
   - show when isPaused = false

3. Pause Screen

   - show when State = PAUSED
   - hide in all other states

   or

   - show when isPaused = true, gameIdle = false
   - hide in all other cases

4. Next Piece

   - hide when State = IDLE, ENDED
   - show in all other states

   or

   - hide when gameIdle = true

5. New Game/End Game

   - show New Game when State = IDLE, ENDED
   - show End Game in all other States

   or

   - show New Game when gameIdle = true
   - show End Game when gameIdle = false
