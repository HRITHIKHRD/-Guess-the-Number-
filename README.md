#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
#include <ctype.h>

#define MAX_TRIES 10
#define MIN_NUMBER 1
#define MAX_NUMBER 200
#define MAX_LEADERBOARD_ENTRIES 5
#define MAX_NAME_LENGTH 50
#define FILE_NAME "leaderboard.txt"
#define GAME_STATE_FILE "game_state.dat"
#define MAX_HINTS 3
#define MAX_PLAYER_STATS 5
#define MAX_ACHIEVEMENTS 5

// Function Declarations
void displayInstructions();
int generateRandomNumber(int lower, int upper);
void playGame(int gameMode, int *totalScore, int *roundsPlayed, char *playerName, int *highScore);
void showScore(int score);
void showLeaderboard();
void addLeaderboardEntry(int score, const char *name);
void displayMainMenu();
void delay(int seconds);
void provideHint(int targetNumber, int guess);
void saveLeaderboardToFile();
void loadLeaderboardFromFile();
void saveGameState(int roundsPlayed, int totalScore, const char *playerName);
void loadGameState(int *roundsPlayed, int *totalScore, char *playerName);
void startNewRound(int *roundsPlayed, int *totalScore);
void displayExitConfirmation();
void displayHighScore(int score);
void clearScreen();
void getPlayerName(char *playerName);
void dailyChallenge(int *totalScore);
void showGameSettings();
void adjustSettings();
void playMultiplayer(char *playerName);
void showPlayerStats();
void simulateSound(const char *message);
void savePlayerStats();
void showAchievements();
void unlockAchievement(const char *achievement);
void loadAchievementsFromFile();
void saveAchievementsToFile();

// Player Statistics Structure
typedef struct {
    char name[MAX_NAME_LENGTH];
    int gamesPlayed;
    int gamesWon;
    int gamesLost;
    int totalScore;
} PlayerStats;

// Achievements Structure
typedef struct {
    char name[MAX_NAME_LENGTH];
    int unlocked;
} Achievement;

// Leaderboard structure
typedef struct {
    char name[MAX_NAME_LENGTH];
    int score;
} LeaderboardEntry;

LeaderboardEntry leaderboard[MAX_LEADERBOARD_ENTRIES];
Achievement achievements[MAX_ACHIEVEMENTS];

// Structure to save game state
typedef struct {
    int roundsPlayed;
    int totalScore;
    int highScore;
    char playerName[MAX_NAME_LENGTH];
} GameState;

int main() {
    int choice;
    int roundsPlayed = 0;
    int totalScore = 0;
    int highScore = 0;
    char playerName[MAX_NAME_LENGTH] = "";

    // Seed the random number generator with current time
    srand(time(NULL));

    // Load leaderboard and achievements from file at the start
    loadLeaderboardFromFile();
    loadAchievementsFromFile();

    // Load previous game state
    loadGameState(&roundsPlayed, &totalScore, playerName);

    // Display game instructions
    displayInstructions();

    // Main menu loop
    while (1) {
        displayMainMenu();
        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                getPlayerName(playerName);
                {
                    int gameMode;
                    printf("\nSelect Game Mode:\n");
                    printf("1. Classic Mode\n");
                    printf("2. Time Challenge Mode\n");
                    printf("3. Score Multiplier Mode\n");
                    printf("4. Multiplayer Mode\n");
                    printf("5. Daily Challenge\n");
                    printf("Enter your choice: ");
                    scanf("%d", &gameMode);
                    playGame(gameMode, &totalScore, &roundsPlayed, playerName, &highScore);
                    break;
                }
            case 2:
                showScore(totalScore);
                break;
            case 3:
                showLeaderboard();
                break;
            case 4:
                saveLeaderboardToFile();
                printf("\nLeaderboard saved!\n");
                break;
            case 5:
                saveGameState(roundsPlayed, totalScore, playerName);
                printf("\nGame saved successfully!\n");
                break;
            case 6:
                loadGameState(&roundsPlayed, &totalScore, playerName);
                printf("\nGame loaded successfully!\n");
                break;
            case 7:
                displayExitConfirmation();
                break;
            case 8:
                displayHighScore(highScore);
                break;
            case 9:
                showPlayerStats();
                break;
            case 10:
                savePlayerStats();
                break;
            case 11:
                showAchievements();
                break;
            default:
                printf("\nInvalid choice, please try again.\n");
        }
    }

    return 0;
}

// Function to display instructions
void displayInstructions() {
    printf("\nWelcome to the 'Guess the Number' Game!\n");
    printf("In this game, you will need to guess a randomly generated number within a specified range.\n");
    printf("You will have a limited number of attempts based on the game mode you choose.\n");
    printf("The closer you guess to the target number, the higher your score will be.\n");
    printf("Good luck!\n\n");
}

// Function to generate a random number within a specified range
int generateRandomNumber(int lower, int upper) {
    return (rand() % (upper - lower + 1)) + lower;
}

// Function to play the game
void playGame(int gameMode, int *totalScore, int *roundsPlayed, char *playerName, int *highScore) {
    int targetNumber, guess, attempts = 0, score = 0;
    int maxAttempts = MAX_TRIES;

    // Select difficulty based on game mode
    switch (gameMode) {
        case 1:  // Classic Mode
            targetNumber = generateRandomNumber(MIN_NUMBER, MAX_NUMBER);
            printf("\nYou are playing Classic Mode.\nGuess a number between 1 and 200.\n");
            break;

        case 2:  // Time Challenge Mode
            targetNumber = generateRandomNumber(MIN_NUMBER, MAX_NUMBER);
            printf("\nYou are playing Time Challenge Mode.\nGuess a number between 1 and 200 within 20 seconds!\n");
            break;

        case 3:  // Score Multiplier Mode
            targetNumber = generateRandomNumber(MIN_NUMBER, MAX_NUMBER);
            printf("\nYou are playing Score Multiplier Mode.\nGuess a number between 1 and 200.\n");
            break;

        case 4:  // Multiplayer Mode
            printf("\nMultiplayer Mode - Compete with another player!\n");
            playMultiplayer(playerName);
            return;  // Exit to avoid duplication
            break;

        case 5:  // Daily Challenge
            dailyChallenge(totalScore);
            return;
            break;

        default:
            printf("\nInvalid Game Mode!\n");
            return;
    }

    // Game logic for all modes except multiplayer and daily challenge
    while (attempts < maxAttempts) {
        printf("\nAttempt %d/%d. Enter your guess: ", attempts + 1, maxAttempts);
        scanf("%d", &guess);

        // Check if the guess is correct
        if (guess == targetNumber) {
            printf("Correct! You've guessed the number %d!\n", targetNumber);
            score = maxAttempts - attempts;  // Higher score for fewer attempts
            *totalScore += score;
            addLeaderboardEntry(score, playerName);
            unlockAchievement("Guess a correct number");
            break;
        } else if (guess < targetNumber) {
            printf("Too low! Try again.\n");
        } else {
            printf("Too high! Try again.\n");
        }

        // Provide hints if game mode allows
        if (gameMode != 4) {  // Exclude Multiplayer
            if (attempts >= MAX_HINTS) {
                printf("Hint: The number is between %d and %d.\n", targetNumber - 10, targetNumber + 10);
            }
        }

        attempts++;
    }

    // If max attempts exhausted, reveal the number
    if (attempts == maxAttempts) {
        printf("\nSorry, you've used all your attempts. The correct number was %d.\n", targetNumber);
    }

    // Update rounds played and check for high score
    (*roundsPlayed)++;
    if (*totalScore > *highScore) {
        *highScore = *totalScore;
    }
}

// Function to show the current score
void showScore(int score) {
    printf("\nTotal score: %d\n", score);
}

// Function to show the leaderboard
void showLeaderboard() {
    printf("\n------ Leaderboard ------\n");
    for (int i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
        if (leaderboard[i].score > 0) {
            printf("%d. %s - %d points\n", i + 1, leaderboard[i].name, leaderboard[i].score);
        }
    }
    printf("\n");
}

// Function to add a leaderboard entry
void addLeaderboardEntry(int score, const char *name) {
    int i;
    for (i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
        if (leaderboard[i].score < score) {
            // Shift entries down to make space for the new score
            for (int j = MAX_LEADERBOARD_ENTRIES - 1; j > i; j--) {
                leaderboard[j] = leaderboard[j - 1];
            }
            leaderboard[i].score = score;
            strncpy(leaderboard[i].name, name, MAX_NAME_LENGTH);
            break;
        }
    }
}

// Function to simulate sound effect
void simulateSound(const char *message) {
    printf("[Sound Effect]: %s\n", message);
}

// Function to save leaderboard to a file
void saveLeaderboardToFile() {
    FILE *file = fopen(FILE_NAME, "w");
    if (file != NULL) {
        for (int i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
            if (leaderboard[i].score > 0) {
                fprintf(file, "%s,%d\n", leaderboard[i].name, leaderboard[i].score);
            }
        }
        fclose(file);
    }
}

// Function to load leaderboard from a file
void loadLeaderboardFromFile() {
    FILE *file = fopen(FILE_NAME, "r");
    if (file != NULL) {
        char name[MAX_NAME_LENGTH];
        int score;
        int i = 0;
        while (fscanf(file, "%49[^,],%d\n", name, &score) != EOF && i < MAX_LEADERBOARD_ENTRIES) {
            strncpy(leaderboard[i].name, name, MAX_NAME_LENGTH);
            leaderboard[i].score = score;
            i++;
        }
        fclose(file);
    }
}

// Function to save achievements to a file
void saveAchievementsToFile() {
    FILE *file = fopen("achievements.txt", "w");
    if (file != NULL) {
        for (int i = 0; i < MAX_ACHIEVEMENTS; i++) {
            fprintf(file, "%s,%d\n", achievements[i].name, achievements[i].unlocked);
        }
        fclose(file);
    }
}

// Function to load achievements from a file
void loadAchievementsFromFile() {
    FILE *file = fopen("achievements.txt", "r");
    if (file != NULL) {
        char name[MAX_NAME_LENGTH];
        int unlocked;
        int i = 0;
        while (fscanf(file, "%49[^,],%d\n", name, &unlocked) != EOF && i < MAX_ACHIEVEMENTS) {
            strncpy(achievements[i].name, name, MAX_NAME_LENGTH);
            achievements[i].unlocked = unlocked;
            i++;
        }
        fclose(file);
    }
}

// Function to show achievements
void showAchievements() {
    printf("\n------ Achievements ------\n");
    for (int i = 0; i < MAX_ACHIEVEMENTS; i++) {
        printf("%s - %s\n", achievements[i].name, achievements[i].unlocked ? "Unlocked" : "Locked");
    }
    printf("\n");
}

// Function to unlock an achievement
void unlockAchievement(const char *achievement) {
    for (int i = 0; i < MAX_ACHIEVEMENTS; i++) {
        if (strcmp(achievements[i].name, achievement) == 0) {
            if (!achievements[i].unlocked) {
                achievements[i].unlocked = 1;
                printf("\nAchievement Unlocked: %s\n", achievements[i].name);
                saveAchievementsToFile();  // Save achievements to file after unlocking
            }
            break;
        }
    }
}

// Function to show player statistics
void showPlayerStats() {
    printf("\n------ Player Stats ------\n");
    for (int i = 0; i < MAX_PLAYER_STATS; i++) {
        if (playerStats[i].gamesPlayed > 0) {
            printf("%s - Games Played: %d, Wins: %d, Losses: %d, Total Score: %d\n",
                playerStats[i].name,
                playerStats[i].gamesPlayed,
                playerStats[i].gamesWon,
                playerStats[i].gamesLost,
                playerStats[i].totalScore);
        }
    }
    printf("\n");
}

// Function to get the player's name
void getPlayerName(char *playerName) {
    printf("\nEnter your name: ");
    scanf("%s", playerName);
}

// Function to clear the screen (platform dependent)
void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}

// Function to confirm exit
void displayExitConfirmation() {
    char choice;
    printf("\nAre you sure you want to exit? (y/n): ");
    scanf(" %c", &choice);
    if (choice == 'y' || choice == 'Y') {
        exit(0);
    }
}

// Function to show high score
void displayHighScore(int score) {
    printf("\nHigh Score: %d\n", score);
}

// Function to simulate a daily challenge
void dailyChallenge(int *totalScore) {
    printf("\nWelcome to the Daily Challenge!\n");
    int dailyScore = rand() % 100 + 1;
    printf("Complete today's challenge and earn %d extra points!\n", dailyScore);
    *totalScore += dailyScore;
    printf("Your total score after completing the challenge: %d\n", *totalScore);
}

// Function to adjust game settings (number range, attempts, etc.)
void adjustSettings() {
    int choice;
    printf("\nSettings Menu\n");
    printf("1. Change Number Range\n");
    printf("2. Change Number of Attempts\n");
    printf("3. Back to Main Menu\n");
    printf("Enter your choice: ");
    scanf("%d", &choice);

    switch (choice) {
        case 1:
            printf("Enter new number range (min, max): ");
            int min, max;
            scanf("%d %d", &min, &max);
            printf("New number range set: %d to %d\n", min, max);
            break;
        case 2:
            printf("Enter new number of attempts: ");
            int attempts;
            scanf("%d", &attempts);
            printf("New number of attempts: %d\n", attempts);
            break;
        case 3:
            return;
        default:
            printf("Invalid option, try again.\n");
    }
}

// Function to simulate multiplayer (simple version)
void playMultiplayer(char *playerName) {
    char opponentName[MAX_NAME_LENGTH];
    int playerGuess, opponentGuess;
    printf("\nEnter your opponent's name: ");
    scanf("%s", opponentName);
    int targetNumber = generateRandomNumber(MIN_NUMBER, MAX_NUMBER);

    printf("\nBoth players must guess the number between %d and %d.\n", MIN_NUMBER, MAX_NUMBER);
    printf("%s, your turn to guess: ", playerName);
    scanf("%d", &playerGuess);

    printf("%s, your turn to guess: ", opponentName);
    scanf("%d", &opponentGuess);

    // Determine winner
    int playerDifference = abs(playerGuess - targetNumber);
    int opponentDifference = abs(opponentGuess - targetNumber);

    if (playerDifference < opponentDifference) {
        printf("\n%s wins! The correct number was %d.\n", playerName, targetNumber);
    } else if (playerDifference > opponentDifference) {
        printf("\n%s wins! The correct number was %d.\n", opponentName, targetNumber);
    } else {
        printf("\nIt's a tie! Both players guessed equally far from the correct number.\n");
    }
}
