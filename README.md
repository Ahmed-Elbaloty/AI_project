
    //Minmax Algorithm
    
#include <FastLED.h>
// #define DEMO_MODE
enum Player { NONE, PLAYER1, PLAYER2 };
static constexpr uint8_t BOARD_SIZE = 3;
static constexpr uint8_t COL0_PIN = 4;
static constexpr uint8_t COL1_PIN = 5;
static constexpr uint8_t COL2_PIN = 6;
static constexpr uint8_t ROW0_PIN = A0;
static constexpr uint8_t ROW1_PIN = A1;
static constexpr uint8_t ROW2_PIN = A2;
static constexpr uint8_t LED_PIN = 9;
static constexpr uint8_t NUM_LEDS = BOARD_SIZE * BOARD_SIZE;
static constexpr uint8_t RAND_PIN = A3;
Player turn = PLAYER1;
bool firstMove = true;
Player board[BOARD_SIZE][BOARD_SIZE];
CRGB leds[NUM_LEDS];
void setup() {
 Serial.begin(9600);
 FastLED.addLeds<WS2811, LED_PIN, RGB>(leds, NUM_LEDS);
 pinMode(COL0_PIN, OUTPUT);
 pinMode(COL1_PIN, OUTPUT);
 pinMode(COL2_PIN, OUTPUT);
 pinMode(ROW0_PIN, INPUT_PULLUP);
 pinMode(ROW1_PIN, INPUT_PULLUP);
 pinMode(ROW2_PIN, INPUT_PULLUP);
 randomSeed(analogRead(RAND_PIN));
}
/*
 Code for polling buttons
*/
void setCol(uint8_t targetCol) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (col != targetCol) {
 digitalWrite(col + COL0_PIN, HIGH);
 } else {
 digitalWrite(col + COL0_PIN, LOW);
 }
 }
}
int8_t getRow() {
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 if (digitalRead(row + ROW0_PIN) == LOW) {
 return row;
 }
 }
 return -1;
}
/*
 Code for displaying board
*/
void drawBoard() {
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 switch (board[row][col]) {
 case NONE:
 leds[row + col * BOARD_SIZE] = CRGB::Black;
 break;
 case PLAYER1:
 leds[row + col * BOARD_SIZE] = CRGB::Red;
 break;
 case PLAYER2:
 leds[row + col * BOARD_SIZE] = CRGB::Blue;
 break;
 }
 }
 }
 FastLED.show();
}
/*
 AI
*/
int16_t evaluate() {
 // evaluate diagonals
 if (board[0][0] == board[1][1] && board[1][1] == board[2][2]) {
 if (board[0][0] == PLAYER1) {
 return -10;
 } else if (board[0][0] == PLAYER2) {
 return 10;
 }
 }
 if (board[2][0] == board[1][1] && board[1][1] == board[0][2]) {
 if (board[2][0] == PLAYER1) {
 return -10;
 } else if (board[2][0] == PLAYER2) {
 return 10;
 }
 }
 // evaluate rows
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 if (board[row][0] == board[row][1] && board[row][1] == board[row][2]) {
 if (board[row][0] == PLAYER1) {
 return -10;
 } else if (board[row][0] == PLAYER2) {
 return 10;
 }
 }
 }
 // evaluate columns
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (board[0][col] == board[1][col] && board[1][col] == board[2][col]) {
 if (board[0][col] == PLAYER1) {
 return -10;
 } else if (board[0][col] == PLAYER2) {
 return 10;
 }
 }
 }
 return 0;
}
bool isMoveLeft() {
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (board[row][col] == NONE) {
 return true;
 }
 }
 }
 return false;
}
int16_t miniMax(Player player) {
 int16_t val = evaluate();
 if (val == -10 || val == 10) {
 // player or computer has won
 return val;
 } else if (!isMoveLeft()) {
 return 0;
 }
 if (player == PLAYER2) {
 // maximize
 int16_t bestVal = -10000;
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (board[row][col] != NONE) {
 continue;
 }
 // make move
 board[row][col] = PLAYER2;
 int16_t val = miniMax(PLAYER1);
 bestVal = max(bestVal, val);
 // undo move
 board[row][col] = NONE;
 }
 }
 return bestVal;
 } else {
 // minimize
 int16_t bestVal = 10000;
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (board[row][col] != NONE) {
 continue;
 }
 // make move
 board[row][col] = PLAYER1;
 int16_t val = miniMax(PLAYER2);
 bestVal = min(bestVal, val);
 // undo move
 board[row][col] = NONE;
 }
 }
 return bestVal;
 }
}
void findBestMove(uint8_t& bestRow, uint8_t& bestCol, int16_t& bestVal, Player player) {
 bestVal = player == PLAYER1 ? 10000 : -10000;
 Player nextPlayer = player == PLAYER1 ? PLAYER2 : PLAYER1;
 Serial.println();
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 if (board[row][col] != NONE) {
 continue;
 }
 // make move
 board[row][col] = player;
 int16_t moveVal = miniMax(nextPlayer);
 Serial.print("evaluate ");
 Serial.print(row);
 Serial.print(" / ");
 Serial.print(col);
 Serial.print(": ");
 Serial.println(moveVal);
 // undo move
 board[row][col] = NONE;
 bool isBetter = player == PLAYER1 ? moveVal < bestVal : moveVal > bestVal;
 if (isBetter) {
 bestRow = row;
 bestCol = col;
 bestVal = moveVal;
 }
 if ((player == PLAYER1 && bestVal == -10) || (player == PLAYER2 && bestVal == 10)) {
 return;
 }
 }
 }
}
/*
 ANIMATIONS
*/
void animation(CRGB color) {
 for (int i = 0; i < NUM_LEDS; ++i) {
 for (int j = 0; j < NUM_LEDS; ++j) {
 if (j == i) {
 leds[j] = color;
 } else {
 leds[j] = CRGB::Black;
 }
 }
 FastLED.show();
 delay(200);
 }
 for (int brightness = 255; brightness >= 0; --brightness) {
 for (int i = 0; i < NUM_LEDS; ++i) {
 FastLED.setBrightness(brightness);
 leds[i] = color;
 }
 FastLED.show();
 delay(10);
 }
}
void debugBoard() {
 Serial.println();
 for (uint8_t row = 0; row < BOARD_SIZE; ++row) {
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 Serial.print(board[row][col]);
 Serial.print(" ");
 }
 Serial.println();
 }
 Serial.println();
}
void loop() {
 uint8_t bestRow = 0, bestCol = 0;
 int16_t bestVal = 0;
 if (turn == PLAYER1) {
 if (firstMove) {
 bestRow = random(0, 3);
 bestCol = random(0, 3);
 firstMove = false;
 } else {
 findBestMove(bestRow, bestCol, bestVal, PLAYER1);
 }
 delay(1000);
 board[bestRow][bestCol] = PLAYER1;
 turn = PLAYER2;
 } else {
#ifdef DEMO_MODE
 if (firstMove) {
 bestRow = random(0, 3);
 bestCol = random(0, 3);
 firstMove = false;
 } else {
 findBestMove(bestRow, bestCol, bestVal, PLAYER2);
 }
 delay(1000);
 board[bestRow][bestCol] = PLAYER2;
 turn = PLAYER1;
#else
 for (uint8_t col = 0; col < BOARD_SIZE; ++col) {
 setCol(col);
 int8_t row = getRow();
 if (row != -1) {
 board[row][col] = PLAYER2;
 turn = PLAYER1;
 }
 }
#endif
 }
 drawBoard();
 int16_t val = evaluate();
 if (val == 10) {
 delay(500);
 Serial.println("YOU WON");
 animation(CRGB::Red);
 while (true) {}
 } else if (val == -10) {
 delay(500);
 Serial.println("ARDUINO WON");
 animation(CRGB::Blue);
 while (true) {}
 } else if (val == 0 && !isMoveLeft()) {
 delay(500);
 Serial.println("DRAW");
 animation(CRGB::Green);
 while (true) {}
 }
}
