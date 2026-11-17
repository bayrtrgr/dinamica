/*
 * Problema 1: Escaleras
 * ---------------------------------------------------------------
 * Un personaje sube una escalera de N escalones avanzando 1 o 2
 * escalones por paso. Se pide contar de cuántas formas distintas
 * puede llegar exactamente al escalón N.
 *
 * Estrategia: Programación Dinámica 
 *
 *   Estado DP:
 *     dp[i] = número de formas distintas de llegar exactamente
 *             al escalón i (0 <= i <= N).
 *
 *   Relación de recurrencia:
 *     dp[i] = dp[i-1] + dp[i-2]      para i >= 2
 *     (el último paso fue de 1 escalón desde i-1, o de 2 desde i-2)
 *
 *   Casos base:
 *     dp[0] = 1   (hay una única forma de "no moverse": estar en el suelo)
 *     dp[1] = 1   (solo se puede llegar con un paso de 1 escalón)
 *
 *   Respuesta: dp[N]
 *
 * Complejidad:
 *   Tiempo:  O(N)  -> se llena cada celda de la tabla una sola vez.
 *   Espacio: O(N)  -> se guarda la tabla completa (se requiere mostrarla).
 *
 * Límite numérico: dp[N] = Fibonacci(N+1). Con unsigned long long el
 * mayor N que cabe sin desbordar es 92, por eso se valida la entrada.
 *
 * Compilar: g++ -std=c++17 -O2 -Wall -o escaleras escaleras.cpp
 * Ejecutar: ./escaleras          (pide N por teclado)
 *           ./escaleras 7        (N como argumento)
 */

#include <iostream>
#include <vector>
#include <string>
#include <cstdlib>

using namespace std;

// Mayor N para el que dp[N] cabe en un unsigned long long (Fib(93) < 2^64).
const int N_MAXIMO = 92;

/*
 * Construye y devuelve la tabla DP para N escalones.
 * dp[i] = cantidad de formas de llegar al escalón i.
 */
vector<unsigned long long> construirTabla(int N) {
    vector<unsigned long long> dp(N + 1);

    // Caso base: una forma de estar en el escalón 0 (no hacer nada).
    dp[0] = 1;

    // Caso base: una forma de llegar al escalón 1 (un paso de 1).
    if (N >= 1) dp[1] = 1;

    // Recurrencia: se llega a i desde i-1 (paso de 1) o desde i-2 (paso de 2).
    for (int i = 2; i <= N; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp;
}

/* Imprime la tabla con el formato [a, b, c, ...]. */
void imprimirTabla(const vector<unsigned long long>& dp) {
    cout << "[";
    for (size_t i = 0; i < dp.size(); i++) {
        if (i > 0) cout << ", ";
        cout << dp[i];
    }
    cout << "]" << endl;
}

int main(int argc, char* argv[]) {
    string texto;

    // El valor de N puede venir como argumento o por teclado.
    if (argc > 1) {
        texto = argv[1];
    } else {
        cout << "Ingrese el numero de escalones N: ";
        if (!(cin >> texto)) {
            cerr << "Error: no se pudo leer la entrada." << endl;
            return 1;
        }
    }

    // Validación: debe ser un entero no negativo.
    char* fin = nullptr;
    long long valor = strtoll(texto.c_str(), &fin, 10);
    if (texto.empty() || *fin != '\0') {
        cerr << "Error: '" << texto << "' no es un numero entero valido." << endl;
        return 1;
    }
    if (valor < 0) {
        cerr << "Error: N no puede ser negativo." << endl;
        return 1;
    }
    if (valor > N_MAXIMO) {
        cerr << "Error: N debe ser <= " << N_MAXIMO
             << " para que el resultado quepa en 64 bits." << endl;
        return 1;
    }

    int N = static_cast<int>(valor);
    vector<unsigned long long> dp = construirTabla(N);

    cout << "Formas posibles: " << dp[N] << endl;
    cout << "Tabla DP:" << endl;
    imprimirTabla(dp);
    return 0;
}

/*
 * Problema 2: Cambio minimo de monedas
 * ---------------------------------------------------------------
 * Dado un conjunto de denominaciones (cada una con cantidad
 * ilimitada) y una cantidad objetivo, se pide el numero MINIMO de
 * monedas que suman exactamente esa cantidad, y una combinacion
 * valida que lo logre.
 *
 * Estrategia: Programacion Dinamica (bottom-up / tabulacion).
 *
 *   Estado DP:
 *     dp[a]   = minimo de monedas necesarias para formar exactamente
 *               la cantidad a (0 <= a <= cantidad objetivo).
 *               Si a es imposible de formar, dp[a] = INF.
 *     ult[a]  = ultima moneda usada en una solucion optima de a
 *               (sirve para reconstruir la combinacion).
 *
 *   Relacion de recurrencia:
 *     dp[a] = min sobre cada moneda c <= a, con dp[a-c] != INF, de
 *             ( dp[a - c] + 1 )
 *     (se elige la moneda c como ultima y se resuelve a - c optimamente)
 *
 *   Casos base:
 *     dp[0] = 0     (para formar 0 no se necesita ninguna moneda)
 *     dp[a] = INF   inicialmente para a > 0 (aun no se sabe formarlo)
 *
 *   Respuesta: dp[objetivo]  (si es INF, no existe combinacion).
 *
 *   Reconstruccion: desde a = objetivo se toma ult[a], se resta de a
 *   y se repite hasta llegar a 0.
 *
 * Complejidad (n = cantidad de denominaciones, M = cantidad objetivo):
 *   Tiempo:  O(n * M)  -> para cada cantidad se prueban las n monedas.
 *   Espacio: O(M)      -> arreglos dp y ult de tamano M + 1.
 *
 * Compilar: g++ -std=c++17 -O2 -Wall -o monedas monedas.cpp
 * Ejecutar: ./monedas
 *   Formato de entrada (por teclado o redirigido):
 *     n                      (cantidad de denominaciones)
 *     c1 c2 ... cn           (valores de las monedas, enteros > 0)
 *     M                      (cantidad objetivo, entero >= 0)
 *   Ejemplo:  echo "3  1 3 4  6" | ./monedas
 */

#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>
#include <set>

using namespace std;

// Valor "infinito": marca las cantidades que no se pueden formar.
const int INF = INT_MAX;

// Si la cantidad objetivo es mayor, no se imprime la tabla completa.
const int LIMITE_TABLA = 200;

// Si la combinacion tiene mas monedas que esto, se imprime agrupada (valor xcantidad).
const size_t LIMITE_COMBINACION = 30;

/*
 * Resultado del algoritmo: tabla de minimos y ultima moneda usada.
 */
struct Resultado {
    vector<int> dp;   // dp[a] = minimo de monedas para a (o INF)
    vector<int> ult;  // ult[a] = ultima moneda de una solucion optima de a
};

/*
 * Llena la tabla DP para las monedas dadas y la cantidad objetivo.
 */
Resultado resolver(const vector<int>& monedas, int objetivo) {
    Resultado r;
    r.dp.assign(objetivo + 1, INF);
    r.ult.assign(objetivo + 1, 0);

    // Caso base: con 0 monedas se forma la cantidad 0.
    r.dp[0] = 0;

    // Se calcula cada cantidad de menor a mayor (los subproblemas
    // dp[a - c] ya estan resueltos cuando se necesitan).
    for (int a = 1; a <= objetivo; a++) {
        for (int c : monedas) {
            // La moneda c solo sirve si cabe en a y a - c es formable.
            if (c <= a && r.dp[a - c] != INF) {
                int candidato = r.dp[a - c] + 1;
                if (candidato < r.dp[a]) {
                    r.dp[a] = candidato;
                    r.ult[a] = c;   // recordamos la moneda elegida
                }
            }
        }
    }
    return r;
}

/*
 * Reconstruye una combinacion optima siguiendo ult[] desde el objetivo.
 * Se devuelve ordenada de mayor a menor para una lectura mas clara.
 */
vector<int> reconstruir(const Resultado& r, int objetivo) {
    vector<int> usadas;
    int a = objetivo;
    while (a > 0) {
        int c = r.ult[a];
        usadas.push_back(c);
        a -= c;
    }
    sort(usadas.rbegin(), usadas.rend());
    return usadas;
}

/* Imprime la tabla con el formato [a, b, c, ...] (INF si no es formable). */
void imprimirTabla(const vector<int>& dp) {
    cout << "[";
    for (size_t i = 0; i < dp.size(); i++) {
        if (i > 0) cout << ",";
        if (dp[i] == INF) cout << "INF";
        else cout << dp[i];
    }
    cout << "]" << endl;
}

int main() {
    int n;
    cout << "Ingrese la cantidad de denominaciones: ";
    if (!(cin >> n) || n <= 0) {
        cerr << "Error: la cantidad de denominaciones debe ser un entero positivo." << endl;
        return 1;
    }

    // Se leen las monedas; se validan (>0) y se eliminan duplicados.
    cout << "Ingrese los " << n << " valores de las monedas: ";
    set<int> unicas;
    for (int i = 0; i < n; i++) {
        int c;
        if (!(cin >> c)) {
            cerr << "Error: entrada de moneda invalida." << endl;
            return 1;
        }
        if (c <= 0) {
            cerr << "Error: cada moneda debe ser un entero positivo (recibido " << c << ")." << endl;
            return 1;
        }
        unicas.insert(c);
    }
    vector<int> monedas(unicas.begin(), unicas.end());

    int objetivo;
    cout << "Ingrese la cantidad objetivo: ";
    if (!(cin >> objetivo) || objetivo < 0) {
        cerr << "Error: la cantidad objetivo debe ser un entero >= 0." << endl;
        return 1;
    }

    cout << endl;
    cout << "Monedas: [";
    for (size_t i = 0; i < monedas.size(); i++) {
        if (i > 0) cout << ",";
        cout << monedas[i];
    }
    cout << "]" << endl;
    cout << "Cantidad: " << objetivo << endl;

    Resultado r = resolver(monedas, objetivo);

    if (r.dp[objetivo] == INF) {
        cout << "No es posible formar la cantidad " << objetivo
             << " con las monedas dadas." << endl;
    } else {
        cout << "Cantidad minima de monedas: " << r.dp[objetivo] << endl;

        // Combinacion valida (vacia si la cantidad es 0).
        vector<int> comb = reconstruir(r, objetivo);
        cout << "Combinacion: ";
        if (comb.empty()) {
            cout << "(ninguna moneda)";
        } else if (comb.size() <= LIMITE_COMBINACION) {
            // Combinacion corta: se imprime moneda por moneda.
            for (size_t i = 0; i < comb.size(); i++) {
                if (i > 0) cout << " + ";
                cout << comb[i];
            }
        } else {
            // Combinacion larga: se agrupan las monedas repetidas (valor x cantidad).
            bool primero = true;
            for (size_t i = 0; i < comb.size();) {
                size_t j = i;
                while (j < comb.size() && comb[j] == comb[i]) j++;
                if (!primero) cout << " + ";
                cout << comb[i] << " x" << (j - i);
                primero = false;
                i = j;
            }
        }
        cout << endl;
    }

    cout << "Tabla DP:" << endl;
    if (objetivo <= LIMITE_TABLA) {
        imprimirTabla(r.dp);
    } else {
        cout << "(tabla omitida: la cantidad supera " << LIMITE_TABLA
             << " y seria demasiado larga)" << endl;
    }
    return 0;
}

