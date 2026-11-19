#include <iostream>
#include <vector>
#include <string>
#include <iomanip>
#include <limits>
#include <algorithm>
#include <stdexcept>
#include <cctype>
#include <clocale>

using namespace std;

const long long INF = numeric_limits<long long>::max() / 4;
const long long MAX_WEIGHT = 1000000000000LL;

// Перевірка позначення нескінченності
bool isInfinity(const string& text) {
    string value = text;

    transform(value.begin(), value.end(), value.begin(),
        [](unsigned char c) {
            return static_cast<char>(tolower(c));
        });

    return value == "inf" || value == "infinity" || value == "x";
}

// Перевірка правильності числа
bool parseNumber(const string& text, long long& number) {
    try {
        size_t position = 0;
        number = stoll(text, &position);

        if (position != text.size()) return false;
        if (number < -MAX_WEIGHT || number > MAX_WEIGHT)
            return false;

        return true;
    }
    catch (...) {
        return false;
    }
}

// Захист від переповнення
long long addDistance(long long a, long long b) {
    if (b > 0 && a > INF - b) return INF;
    if (b < 0 && a < -INF - b) return -INF;

    return a + b;
}

// Алгоритм Флойда–Уоршелла
void floydWarshall(vector<vector<long long>>& dist,
    vector<vector<int>>& nextVertex) {

    int n = dist.size();

    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {

            if (dist[i][k] == INF) continue;

            for (int j = 0; j < n; j++) {

                if (dist[k][j] == INF) continue;

                long long newDistance =
                    addDistance(dist[i][k], dist[k][j]);

                if (newDistance < dist[i][j]) {
                    dist[i][j] = newDistance;
                    nextVertex[i][j] = nextVertex[i][k];
                }
            }
        }
    }
}

// Отримання маршруту
string getPath(int from, int to,
    const vector<vector<int>>& nextVertex) {

    int n = nextVertex.size();

    if (nextVertex[from][to] == -1)
        return "Немає шляху";

    vector<int> path;

    int current = from;
    path.push_back(current);

    for (int step = 0; current != to && step < n; step++) {

        current = nextVertex[current][to];

        if (current == -1)
            return "Немає шляху";

        path.push_back(current);
    }

    if (current != to)
        return "Помилка маршруту";

    string result;

    for (int i = 0; i < path.size(); i++) {
        if (i > 0) result += "->";

        result += to_string(path[i] + 1);
    }

    return result;
}

// Виведення матриці відстаней
void printDistanceMatrix(
    const vector<vector<long long>>& dist) {

    int n = dist.size();

    cout << "\nМатриця мiнiмальних вiдстаней:\n";

    cout << setw(8) << " ";

    for (int j = 0; j < n; j++)
        cout << setw(8) << j + 1;

    cout << '\n';

    for (int i = 0; i < n; i++) {

        cout << setw(8) << i + 1;

        for (int j = 0; j < n; j++) {

            if (dist[i][j] == INF)
                cout << setw(8) << "INF";
            else
                cout << setw(8) << dist[i][j];
        }

        cout << '\n';
    }
}

// Виведення матриці маршрутів
void printPathMatrix(
    const vector<vector<int>>& nextVertex) {

    int n = nextVertex.size();

    cout << "\nМатриця наступних вершин:\n";

    cout << setw(8) << " ";

    for (int j = 0; j < n; j++)
        cout << setw(8) << j + 1;

    cout << '\n';

    for (int i = 0; i < n; i++) {

        cout << setw(8) << i + 1;

        for (int j = 0; j < n; j++) {

            if (nextVertex[i][j] == -1)
                cout << setw(8) << "-";
            else
                cout << setw(8) << nextVertex[i][j] + 1;
        }

        cout << '\n';
    }
}

int main() {

    setlocale(LC_ALL, "Ukrainian");

    int n;

    cout << "Введiть кiлькiсть вершин графа (вiд 1 до 100): ";

    if (!(cin >> n) || n < 1 || n > 100) {
        cout << "Помилка: некоректна кiлькiсть вершин.\n";
        return 1;
    }

    vector<vector<long long>> dist(
        n, vector<long long>(n, INF));

    vector<vector<int>> nextVertex(
        n, vector<int>(n, -1));

    cout << "\nВведiть матрицю вiдстаней.\n";
    cout << "Для позначення вiдсутнього ребра введiть INF.\n";
    cout << "Головна дiагональ повинна складатися з нулiв.\n\n";

    // Введення матриці
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {

            string token;

            if (!(cin >> token)) {
                cout << "Помилка: не вдалося прочитати матрицю.\n";
                return 1;
            }

            if (isInfinity(token)) {

                if (i == j) {
                    cout << "Помилка: на головнiй дiагоналi мають бути нулi.\n";
                    return 1;
                }

                dist[i][j] = INF;
                nextVertex[i][j] = -1;
            }
            else {

                long long value;

                if (!parseNumber(token, value)) {
                    cout << "Помилка: некоректне значення "
                        << token << ".\n";
                    return 1;
                }

                if (i == j && value != 0) {
                    cout << "Помилка: на головнiй дiагоналi мають бути нулi.\n";
                    return 1;
                }

                dist[i][j] = value;
                nextVertex[i][j] = j;
            }
        }
    }

    // Виконання алгоритму
    floydWarshall(dist, nextVertex);

    // Перевірка на від'ємні цикли
    for (int i = 0; i < n; i++) {

        if (dist[i][i] < 0) {

            cout << "\nПомилка: граф мiстить цикл вiд'ємної ваги.\n";
            cout << "Найкоротшi шляхи не мають скiнченного значення.\n";

            return 1;
        }
    }

    int choice;

    // Головне меню
    while (true) {

        cout << "\n===== МЕНЮ =====\n";
        cout << "1 - Знайти шляхи для всiх пар вершин\n";
        cout << "2 - Знайти шлях мiж двома вершинами\n";
        cout << "0 - Вихiд\n";
        cout << "Ваш вибiр: ";

        if (!(cin >> choice)) {

            cout << "Помилка: потрiбно ввести число.\n";

            if (cin.eof()) break;

            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');

            continue;
        }

        if (choice == 0) {

            cout << "Завершення програми.\n";
            break;
        }

        // Пошук для всіх вершин
        else if (choice == 1) {

            printDistanceMatrix(dist);
            printPathMatrix(nextVertex);
        }

        // Пошук між двома вершинами
        else if (choice == 2) {

            int from, to;

            cout << "Введiть початкову та кiнцеву вершини (вiд 1 до "
                << n << "): ";

            if (!(cin >> from >> to)) {

                cout << "Помилка: потрiбно ввести два числа.\n";

                if (cin.eof()) break;

                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');

                continue;
            }

            if (from < 1 || from > n || to < 1 || to > n) {

                cout << "Помилка: таких вершин у графi немає.\n";
                continue;
            }

            from--;
            to--;

            if (dist[from][to] == INF) {

                cout << "Шляху мiж цими вершинами немає.\n";
            }
            else {

                cout << "Найкоротша вiдстань: "
                    << dist[from][to] << '\n';

                cout << "Маршрут: "
                    << getPath(from, to, nextVertex) << '\n';
            }
        }

        else {

            cout << "Помилка: виберіть 0, 1 або 2.\n";
        }
    }

    return 0;
}

