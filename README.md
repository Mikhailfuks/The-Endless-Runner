#include <SFML/Graphics.hpp>
#include <iostream>
#include <random>

using namespace sf;

// Класс для игрока
class Player {
public:
    Sprite sprite; // Спрайт игрока
    float speed = 5.0f; // Скорость движения
    float jumpHeight = 10.0f; // Высота прыжка
    float gravity = 0.5f; // Сила гравитации
    bool isJumping = false; // Флаг прыжка
    float jumpVelocity = 0.0f; // Скорость прыжка

    // Инициализация игрока
    void init(Texture& texture) {
        sprite.setTexture(texture);
        sprite.setOrigin(sprite.getTextureRect().width / 2, sprite.getTextureRect().height / 2);
        sprite.setPosition(100.0f, 400.0f);
    }

    // Обновление положения игрока
    void update(float deltaTime) {
        if (isJumping) {
            sprite.move(0.0f, -jumpVelocity * deltaTime);
            jumpVelocity += gravity * deltaTime;
            if (sprite.getPosition().y >= 400.0f) {
                sprite.setPosition(sprite.getPosition().x, 400.0f);
                isJumping = false;
                jumpVelocity = 0.0f;
            }
        } else {
            sprite.move(speed * deltaTime, 0.0f);
        }
    }

    // Прыжок игрока
    void jump() {
        if (!isJumping) {
            isJumping = true;
            jumpVelocity = -jumpHeight;
        }
    }
};

// Класс для препятствий
class Obstacle {
public:
    Sprite sprite; // Спрайт препятствия
    float speed = -5.0f; // Скорость движения препятствия

    // Инициализация препятствия
    void init(Texture& texture) {
        sprite.setTexture(texture);
        sprite.setOrigin(sprite.getTextureRect().width / 2, sprite.getTextureRect().height / 2);
        sprite.setPosition(800.0f, 400.0f);
    }

    // Обновление положения препятствия
    void update(float deltaTime) {
        sprite.move(speed * deltaTime, 0.0f);
        if (sprite.getPosition().x < 0) {
            // Перемещение препятствия вправо за экран
            sprite.setPosition(800.0f, 400.0f);
        }
    }

    // Проверка столкновения с игроком
    bool isColliding(Player& player) {
        return sprite.getGlobalBounds().intersects(player.sprite.getGlobalBounds());
    }
};

int main() {
    // Инициализация окна
    RenderWindow window(VideoMode(800, 600), "Бесконечный бегун");

    // Загрузка текстур
    Texture playerTexture;
    if (!playerTexture.loadFromFile("player.png")) {
        cerr << "Ошибка загрузки текстуры игрока\n";
        return 1;
    }

    Texture obstacleTexture;
    if (!obstacleTexture.loadFromFile("obstacle.png")) {
        cerr << "Ошибка загрузки текстуры препятствия\n";
        return 1;
    }

    // Создание игрока и препятствий
    Player player;
    player.init(playerTexture);

    vector<Obstacle> obstacles;
    obstacles.push_back(Obstacle());
    obstacles.back().init(obstacleTexture);

    // Генератор случайных чисел
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<> dist(100, 300);

    // Цикл игры
    Clock clock;
    while (window.isOpen()) {
        float deltaTime = clock.restart().asSeconds();

        // Обработка событий
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }
            if (event.type == Event::KeyPressed) {
                if (event.key.code == Keyboard::Space) {
                    player.jump();
                }
            }
        }

        // Обновление положения игрока и препятствий
        player.update(deltaTime);
        for (auto& obstacle : obstacles) {
            obstacle.update(deltaTime);
            // Проверка столкновения с игроком
            if (obstacle.isColliding(player)) {
                cout << "Столкновение!\n";
                // Обработка столкновения
                // ...
            }
        }

        // Генерация новых препятствий
        if (obstacles.back().sprite.getPosition().x < 400.0f) {
            obstacles.push_back(Obstacle());
            obstacles.back().init(obstacleTexture);
            obstacles.back().sprite.setPosition(800.0f, 400.0f - dist(gen));
        }

        // Очистка экрана
        window.clear(Color::White);

        // Отрисовка игрока и препятствий
        window.draw(player.sprite);
        for (const auto& obstacle : obstacles) {
            window.draw(obstacle.sprite);
        }

        // Отображение окна
        window.display();
    }

    return 0;
}
