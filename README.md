using Raylib_cs;

namespace SnakeGame;

public static class Program
{
    private const int CellSize = 20;
    private const int GridWidth = 30;
    private const int GridHeight = 20;
    private const int WindowWidth = GridWidth * CellSize;
    private const int WindowHeight = GridHeight * CellSize;

    public static void Main()
    {
        Raylib.InitWindow(WindowWidth, WindowHeight, "Snake Game");
        Raylib.SetTargetFPS(10);

        var snake = new List<(int X, int Y)>
        {
            (10, 10),
            (9, 10),
            (8, 10)
        };

        var direction = (X: 1, Y: 0);
        var nextDirection = direction;
        var food = GenerateFood(snake);
        var score = 0;
        var gameOver = false;

        while (!Raylib.WindowShouldClose())
        {
            if (!gameOver)
            {
                UpdateInput(ref nextDirection, direction);
                direction = nextDirection;

                var head = snake[0];
                var newHead = (head.X + direction.X, head.Y + direction.Y);

                if (newHead.X < 0 || newHead.X >= GridWidth ||
                    newHead.Y < 0 || newHead.Y >= GridHeight ||
                    snake.Contains(newHead))
                {
                    gameOver = true;
                }
                else
                {
                    snake.Insert(0, newHead);

                    if (newHead == food)
                    {
                        score++;
                        food = GenerateFood(snake);
                    }
                    else
                    {
                        snake.RemoveAt(snake.Count - 1);
                    }
                }
            }
            else if (Raylib.IsKeyPressed(KeyboardKey.KEY_ENTER) || Raylib.IsKeyPressed(KeyboardKey.KEY_R))
            {
                ResetGame(out snake, out direction, out nextDirection, out food, out score, out gameOver);
            }

            Raylib.BeginDrawing();
            Raylib.ClearBackground(Color.RAYWHITE);

            DrawGrid();
            DrawFood(food);
            DrawSnake(snake);
            DrawScore(score);

            if (gameOver)
            {
                Raylib.DrawText("Game Over", WindowWidth / 2 - 120, WindowHeight / 2 - 40, 40, Color.RED);
                Raylib.DrawText("Press ENTER to restart", WindowWidth / 2 - 170, WindowHeight / 2 + 20, 24, Color.DARKGRAY);
            }

            Raylib.EndDrawing();
        }

        Raylib.CloseWindow();
    }

    private static void UpdateInput(ref (int X, int Y) nextDirection, (int X, int Y) direction)
    {
        if (Raylib.IsKeyPressed(KeyboardKey.KEY_UP) && direction != (0, 1))
            nextDirection = (0, -1);
        else if (Raylib.IsKeyPressed(KeyboardKey.KEY_DOWN) && direction != (0, -1))
            nextDirection = (0, 1);
        else if (Raylib.IsKeyPressed(KeyboardKey.KEY_LEFT) && direction != (1, 0))
            nextDirection = (-1, 0);
        else if (Raylib.IsKeyPressed(KeyboardKey.KEY_RIGHT) && direction != (-1, 0))
            nextDirection = (1, 0);
    }

    private static (int X, int Y) GenerateFood(List<(int X, int Y)> snake)
    {
        var random = new Random();
        (int X, int Y) food;

        do
        {
            food = (random.Next(0, GridWidth), random.Next(0, GridHeight));
        }
        while (snake.Contains(food));

        return food;
    }

    private static void ResetGame(
        out List<(int X, int Y)> snake,
        out (int X, int Y) direction,
        out (int X, int Y) nextDirection,
        out (int X, int Y) food,
        out int score,
        out bool gameOver)
    {
        snake = new List<(int X, int Y)>
        {
            (10, 10),
            (9, 10),
            (8, 10)
        };

        direction = (1, 0);
        nextDirection = direction;
        food = GenerateFood(snake);
        score = 0;
        gameOver = false;
    }

    private static void DrawGrid()
    {
        for (var x = 0; x < GridWidth; x++)
        {
            for (var y = 0; y < GridHeight; y++)
            {
                Raylib.DrawRectangleLines(x * CellSize, y * CellSize, CellSize, CellSize, Color.LIGHTGRAY);
            }
        }
    }

    private static void DrawSnake(List<(int X, int Y)> snake)
    {
        foreach (var segment in snake)
        {
            Raylib.DrawRectangle(segment.X * CellSize + 1, segment.Y * CellSize + 1, CellSize - 2, CellSize - 2, Color.GREEN);
        }
    }

    private static void DrawFood((int X, int Y) food)
    {
        Raylib.DrawRectangle(food.X * CellSize + 3, food.Y * CellSize + 3, CellSize - 6, CellSize - 6, Color.RED);
    }

    private static void DrawScore(int score)
    {
        Raylib.DrawText($"Score: {score}", 10, 10, 20, Color.DARKGRAY);
    }
}
