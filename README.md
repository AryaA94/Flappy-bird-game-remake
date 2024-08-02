import kaboom from "kaboom";

// Initialize context
kaboom();

// Load assets
loadSprite("birdy", "sprites/birdy.png");
loadSprite("bg", "sprites/bg.jpg");
loadSprite("pipe", "sprites/pipe.png");
loadSound("spring", "sounds/spring.mp3");

// Game constants
const PIPE_GAP = 130;
const PIPE_SPEED = 160;
const JUMP_FORCE = 400;
const GAME_OVER_TEXT_SIZE = 50;
const INTRO_TEXT_SIZE = 50;

// Game state
let highScore = 0;

// Intro scene
scene("intro", () => {
  add([
    text("Press space to start", { size: INTRO_TEXT_SIZE }),
    pos(width() / 2, height() / 2),
    origin("center"),
  ]);

  keyPress("space", () => {
    go("game");
  });
});

// Game scene
scene("game", () => {
  let score = 0;

  // Add background
  add([
    sprite("bg", { width: width(), height: height() }),
  ]);

  // Add score text
  const scoreText = add([
    text(score, { size: 50 }),
  ]);

  // Add player
  const player = add([
    sprite("birdy"),
    scale(0.12),
    pos(80, 40),
    area(),
    body(),
  ]);

  // Function to produce pipes
  function producePipes() {
    const offset = rand(-50, 50);

    // Add top pipe
    add([
      sprite("pipe"),
      pos(width(), height() / 2 + offset + PIPE_GAP / 2),
      "pipe",
      area(),
      { passed: false },
    ]);

    // Add bottom pipe
    add([
      sprite("pipe", { flipY: true }),
      pos(width(), height() / 2 + offset - PIPE_GAP / 2),
      origin("botleft"),
      "pipe",
      area(),
    ]);
  }

  // Loop to produce pipes
  loop(1.5, () => {
    producePipes();
  });

  // Action to move pipes
  action("pipe", (pipe) => {
    pipe.move(-PIPE_SPEED, 0);

    // Check if pipe has passed player
    if (pipe.passed === false && pipe.pos.x < player.pos.x) {
      pipe.passed = true;
      score += 1;
      scoreText.text = score;
    }
  });

  // Collision with pipe
  player.collides("pipe", () => {
    go("gameover", score);
  });

  // Check for out of bounds
  player.action(() => {
    if (player.pos.y > height() + 30 || player.pos.y < -30) {
      go("gameover", score);
    }
  });

  // Jump on space press
  keyPress("space", () => {
    play("spring");
    player.jump(JUMP_FORCE);
  });
});

// Game over scene
scene("gameover", (score) => {
  if (score > highScore) {
    highScore = score;
  }

  // Add game over text
  add([
    text(
      "Game Over\n"
      + "Press space to try again\n"
      + "Score: " + score
      + "\nHigh Score: " + highScore,
      { size: GAME_OVER_TEXT_SIZE }
    ),
  ]);

  // Restart game on space press
  keyPress("space", () => {
    go("game");
  });
});

// Start intro
go("intro");


// Sprites (images) and sound effects were either found on the internet or were made by myself
