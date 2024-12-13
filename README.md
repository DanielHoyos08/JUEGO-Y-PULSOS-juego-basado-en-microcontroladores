# JUEGO-Y-PULSOS-juego-basado-en-microcontroladores
Este programa implementa un sistema interactivo que permite controlar el movimiento de un jugador en una pantalla TFT utilizando botones físicos conectados a un microcontrolador. La pantalla tiene una resolución de 240x320 píxeles y se usa para mostrar una animación de un jugador que se mueve en las cuatro direcciones cardinales,
#include "SPI.h"
#include "Adafruit_GFX.h"
#include "Adafruit_ILI9341.h"
#include "Sprite.h"

// Pines del TFT
#define TFT_DC 7  
#define TFT_CS 6
#define TFT_MOSI 11
#define TFT_CLK 13
#define TFT_RST 10
#define TFT_MISO 12

// Pines de los botones
#define botonRight 18
#define botonLeft 21
#define botonUp 19
#define botonDown 20

const int XMAX = 240; // Ancho del display
const int YMAX = 320; // Alto del display
int x = XMAX / 2;      // Posición inicial en x del jugador
int y = YMAX - 40;     // Posición inicial en y del jugador
int lastX = x, lastY = y; // Última posición del jugador
const uint8_t UP = 0;
const uint8_t DOWN = 1;
const uint8_t RIGHT = 2;
const uint8_t LEFT = 3;

#define NUM_OBSTACULOS 5
int obstaculosX[NUM_OBSTACULOS]; // Posiciones X de los obstáculos
int obstaculosY[NUM_OBSTACULOS]; // Posiciones Y de los obstáculos
uint16_t coloresObstaculos[NUM_OBSTACULOS]; // Colores fijos para cada obstáculo

Adafruit_ILI9341 screen = Adafruit_ILI9341(TFT_CS, TFT_DC, TFT_MOSI, TFT_CLK, TFT_RST, TFT_MISO);

Sprite playerSprite(&screen, "player_sprite.bmp", 32, 32); // Ruta de la imagen del jugador y dimensiones
Sprite obstacleSprites[NUM_OBSTACULOS]; // Arreglo de sprites para obstáculos

void setPlayerPosition(int x, int y);
void animatePlayer();
void moverPlayer(uint8_t direccion);
void setupObstaculos();
void moverObstaculos();
void dibujarObstaculos();
bool verificarColision();
void mostrarGameOver();

void setup() {
    Serial.begin(9600);
    Serial.println("Inicializando...");
    attachInterrupt(digitalPinToInterrupt(botonRight), []() { moverPlayer(RIGHT); }, FALLING);
    attachInterrupt(digitalPinToInterrupt(botonLeft), []() { moverPlayer(LEFT); }, FALLING);
    attachInterrupt(digitalPinToInterrupt(botonUp), []() { moverPlayer(UP); }, FALLING);
    attachInterrupt(digitalPinToInterrupt(botonDown), []() { moverPlayer(DOWN); }, FALLING);

    screen.begin();
    screen.fillScreen(ILI9341_BLACK);

    setPlayerPosition(x, y);
    setupObstaculos();
}

void loop() {
    animatePlayer();
    moverObstaculos();
    dibujarObstaculos();

    // Verificamos si hubo una colisión
    if (verificarColision()) {
        mostrarGameOver();  // Si hay colisión, muestra Game Over
    }
}

void setPlayerPosition(int x1, int y1) {
    x = x1;
    y = y1;
}

void animatePlayer() {
    static int count = 0;
    screen.fillRect(lastX, lastY, 32, 32, ILI9341_BLACK);  // Borra la posición anterior del jugador
    playerSprite.setPosition(x, y); // Establece la nueva posición del sprite
    playerSprite.show(); // Muestra el sprite en la pantalla
    lastX = x;
    lastY = y;
    count = (count + 1) % 2;
    delay(50);
}

void moverPlayer(uint8_t direccion) {
    uint8_t delta = 5;
    switch (direccion) {
        case UP: if (y - delta >= 0) y -= delta; break;
        case DOWN: if (y + delta <= YMAX - 32) y += delta; break;
        case RIGHT: if (x + delta <= XMAX - 32) x += delta; break;
        case LEFT: if (x - delta >= 0) x -= delta; break;
    }
    Serial.print("Posición -> X: ");
    Serial.print(x);
    Serial.print(", Y: ");
    Serial.println(y);
}

void setupObstaculos() {
    for (int i = 0; i < NUM_OBSTACULOS; i++) {
        obstaculosX[i] = random(0, XMAX - 20);  // Posición X aleatoria
        obstaculosY[i] = random(-YMAX, 0);      // Empieza fuera de la pantalla
        // Asignamos un color aleatorio para cada obstáculo (entre azul y rojo)
        coloresObstaculos[i] = random(ILI9341_BLUE, ILI9341_RED);
        obstacleSprites[i] = Sprite(&screen, "obstacle_sprite.bmp", 20, 20); // Ruta de la imagen del obstáculo y dimensiones
        obstacleSprites[i].setColor(coloresObstaculos[i]);
        obstacleSprites[i].setPosition(obstaculosX[i], obstaculosY[i]);
    }
}

void moverObstaculos() {
    for (int i = 0; i < NUM_OBSTACULOS; i++) {
        obstaculosY[i] += 5;  // Mover el obstáculo hacia abajo
        if (obstaculosY[i] > YMAX) {  // Si el obstáculo sale de la pantalla
            obstaculosY[i] = random(-YMAX, 0);  // Vuelve a aparecer en la parte superior
            obstaculosX[i] = random(0, XMAX - 20);  // Posición X aleatoria
            obstacleSprites[i].setPosition(obstaculosX[i], obstaculosY[i]);
        } else {
            obstacleSprites[i].setPosition(obstaculosX[i], obstaculosY[i]);
        }
    }
}

void dibujarObstaculos() {
    for (int i = 0; i < NUM_OBSTACULOS; i++) {
        obstacleSprites[i].show(); // Dibuja cada obstáculo en su posición
    }
}

bool verificarColision() {
    for (int i = 0; i < NUM_OBSTACULOS; i++) {
        if (x < obstaculosX[i] + 20 && x + 32 > obstaculosX[i] && 
            y < obstaculosY[i] + 20 && y + 32 > obstaculosY[i]) {
            return true;
        }
    }
    return false;
}

void mostrarGameOver() {
    screen.fillScreen(ILI9341_BLACK);
    screen.setTextColor(ILI9341_WHITE);
    screen.setTextSize(3);
    screen.setCursor(50, YMAX / 2);
    screen.println("GAME OVER");
    delay(3000);
    exit(0); // Detiene el juego
}
