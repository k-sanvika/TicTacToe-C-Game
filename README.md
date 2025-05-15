#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define BOARD_SIZE 3
#define PLAYER 'X'
#define COMPUTER 'O'

typedef struct {
    int player;
    int computer;
    int draw;
} Score;

int gameDifficulty;
Score gameScore = {.player = 0, .computer = 0, .draw = 0};

void input_difficulty();
void clear_screen();
void print_board(char board[BOARD_SIZE][BOARD_SIZE]);
int check_win(char board[BOARD_SIZE][BOARD_SIZE], char symbol);
int check_draw(char board[BOARD_SIZE][BOARD_SIZE]);
void play_game();
void player_move(char board[BOARD_SIZE][BOARD_SIZE]);
void computer_move(char board[BOARD_SIZE][BOARD_SIZE]);
int is_valid_move(char board[BOARD_SIZE][BOARD_SIZE], int row, int col);

int main() {
    srand(time(NULL));
    int playAgain;
    input_difficulty();

    do {
        play_game();
        printf("\nPlay again? (1 for yes, 0 for no): ");
        scanf("%d", &playAgain);
    } while (playAgain == 1);

    printf("\nThank you for playing. Goodbye!\n");
    return 0;
}

void play_game() {
    char board[BOARD_SIZE][BOARD_SIZE] = {
        {' ', ' ', ' '},
        {' ', ' ', ' '},
        {' ', ' ', ' '}
    };

    char currentPlayer = rand() % 2 == 0 ? PLAYER : COMPUTER;
    print_board(board);

    while (1) {
        if (currentPlayer == PLAYER) {
            player_move(board);
            print_board(board);

            if (check_win(board, PLAYER)) {
                gameScore.player++;
                printf("Congratulations! You have won!\n");
                break;
            }
            currentPlayer = COMPUTER;
        } else {
            computer_move(board);
            print_board(board);

            if (check_win(board, COMPUTER)) {
                gameScore.computer++;
                printf("I won! But you played well.\n");
                break;
            }
            currentPlayer = PLAYER;
        }

        if (check_draw(board)) {
            gameScore.draw++;
            printf("\nIt's a draw!\n");
            break;
        }
    }
}

void player_move(char board[BOARD_SIZE][BOARD_SIZE]) {
    int emptyCount = 0, lastRow = -1, lastCol = -1;

    // Check if only one move is left
    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            if (board[i][j] == ' ') {
                emptyCount++;
                lastRow = i;
                lastCol = j;
            }
        }
    }

    if (emptyCount == 1) {
        board[lastRow][lastCol] = PLAYER;
        return;
    }

    int row, col;
    do {
        printf("\nPlayer's turn (X).\nEnter row and column (1-3): ");
        scanf("%d %d", &row, &col);
        row--; col--; // Convert to 0-based index
    } while (!is_valid_move(board, row, col));

    board[row][col] = PLAYER;
}

void computer_move(char board[BOARD_SIZE][BOARD_SIZE]) {
    // 1. Try to win
    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = COMPUTER;
                if (check_win(board, COMPUTER)) return;
                board[i][j] = ' ';
            }
        }
    }

    // 2. Block player's win
    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = PLAYER;
                if (check_win(board, PLAYER)) {
                    board[i][j] = COMPUTER;
                    return;
                }
                board[i][j] = ' ';
            }
        }
    }

    // 3. Smart move in GOD mode
    if (gameDifficulty == 2) {
        if (board[1][1] == ' ') {
            board[1][1] = COMPUTER;
            return;
        }

        int corners[4][2] = {{0,0}, {0,2}, {2,0}, {2,2}};
        for (int i = 0; i < 4; i++) {
            int r = corners[i][0], c = corners[i][1];
            if (board[r][c] == ' ') {
                board[r][c] = COMPUTER;
                return;
            }
        }
    }

    // 4. First available move
    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = COMPUTER;
                return;
            }
        }
    }
}

int is_valid_move(char board[BOARD_SIZE][BOARD_SIZE], int row, int col) {
    return row >= 0 && col >= 0 && row < BOARD_SIZE && col < BOARD_SIZE && board[row][col] == ' ';
}

int check_win(char board[BOARD_SIZE][BOARD_SIZE], char symbol) {
    for (int i = 0; i < BOARD_SIZE; i++) {
        if ((board[i][0] == symbol && board[i][1] == symbol && board[i][2] == symbol) ||
            (board[0][i] == symbol && board[1][i] == symbol && board[2][i] == symbol))
            return 1;
    }

    if ((board[0][0] == symbol && board[1][1] == symbol && board[2][2] == symbol) ||
        (board[0][2] == symbol && board[1][1] == symbol && board[2][0] == symbol))
        return 1;

    return 0;
}

int check_draw(char board[BOARD_SIZE][BOARD_SIZE]) {
    for (int i = 0; i < BOARD_SIZE; i++)
        for (int j = 0; j < BOARD_SIZE; j++)
            if (board[i][j] == ' ')
                return 0;
    return 1;
}

void print_board(char board[BOARD_SIZE][BOARD_SIZE]) {
    clear_screen();
    printf("Score - Player (X): %d, Computer (O): %d, Draws: %d\n", gameScore.player, gameScore.computer, gameScore.draw);
    printf("\nTic-Tac-Toe Board:\n");

    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            printf(" %c ", board[i][j]);
            if (j < BOARD_SIZE - 1) printf("|");
        }
        if (i < BOARD_SIZE - 1) printf("\n---+---+---\n");
    }
    printf("\n");
}

void input_difficulty() {
    while (1) {
        printf("\nSelect difficulty level:");
        printf("\n1. Human (Standard)");
        printf("\n2. God (Hard)");
        printf("\nEnter your choice: ");
        scanf("%d", &gameDifficulty);

        if (gameDifficulty == 1 || gameDifficulty == 2) break;
        else printf("Invalid choice! Please enter 1 or 2.\n");
    }
}

void clear_screen() {
    #ifdef _WIN32
        system("cls");
    #else
        system("clear");
    #endif
}
