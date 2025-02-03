#include <iostream>
#include <vector>
#include <string>
#include <cstdlib>
#include <ctime>
#include <algorithm>

using namespace std;

struct Carta {
    string palo;
    string color;
    int valor;
};

struct Jugador {
    string nombre;
    vector<Carta> mano;
    int puntaje = 0;
};

void mostrarMenu();
void jugar(vector<Jugador> &jugadores, vector<Carta> &mazo);
void mostrarPuntajes(const vector<Jugador> &jugadores);
void mostrarCreditos();
void inicializarMazo(vector<Carta> &mazo);
void repartirCartas(vector<Carta> &mazo, vector<Jugador> &jugadores);
int calcularPuntaje(const vector<Carta> &mano);
void mostrarMano(const vector<Carta> &mano);
string obtenerNombreCarta(int valor);
void barajarMazo(vector<Carta> &mazo);

int main() {
    cout<<"Bienvenidos al Juego de cartas al azar '31'"<<endl;
    srand(time(0)); 
    vector<Jugador> jugadores(3);
    vector<Carta> mazo;
    bool salir = false;
    for (int i = 0; i < jugadores.size(); i++) {
        cout << "Ingrese el nombre del jugador " << i + 1 << ": ";
        cin >> jugadores[i].nombre;
    }
    inicializarMazo(mazo);
    while (!salir) {
        mostrarMenu();
        int opcion;
        cin >> opcion;

        if (cin.fail()) {
            cin.clear();
            cin.ignore(10000, '\n');
            cout << "Por favor, ingrese una opción válida." << endl;
            continue;
        }

        switch (opcion) {
        case 1:
            jugar(jugadores, mazo);
            break;
        case 2:
            mostrarPuntajes(jugadores);
            break;
        case 3:
            mostrarCreditos();
            break;
        case 4:
            cout << "¡Gracias por jugar, vuelva pronto!" << endl;
            salir = true;
            break;
        default:
            cout << "Opción inválida. Inténtelo nuevamente."<<endl;
        }
    }

    return 0;
}

void mostrarMenu() {
    cout << "Menu Principal: "<<endl;
    cout << "1. Jugar"<<endl;
    cout << "2. Mostrar puntajes"<<endl;
    cout << "3. Créditos"<<endl;
    cout << "4. Salir"<<endl;
    cout << "Seleccione una opción: ";
}

void inicializarMazo(vector<Carta> &mazo) {
    const string palos[] = {"Corazón Rojo", "Corazón Negro", "Diamante Rojo", "Trébol Negro"};
    const string colores[] = {"Rojo", "Negro", "Rojo", "Negro"};

    for (int i = 0; i < 4; i++) {
        for (int valor = 1; valor <= 13; valor++) {
            mazo.push_back({palos[i], colores[i], valor});
        }
    }
    barajarMazo(mazo);
}

void barajarMazo(vector<Carta> &mazo) {
    for (size_t i = 0; i < mazo.size(); i++) {
        size_t j = i + rand() % (mazo.size() - i);
        swap(mazo[i], mazo[j]);
    }
}

void repartirCartas(vector<Carta> &mazo, vector<Jugador> &jugadores) {
    for (auto &jugador : jugadores) {
        jugador.mano.clear();
        for (int j = 0; j < 3; j++) {
            jugador.mano.push_back(mazo.back());
            mazo.pop_back();
        }
    }
}

int calcularPuntaje(const vector<Carta> &mano) {
    int maxValor = 0;
    for (const auto &carta : mano) {
        maxValor = max(maxValor, carta.valor);
    }
    return maxValor;
}

void mostrarMano(const vector<Carta> &mano) {
    for (const auto &carta : mano) {
        cout << obtenerNombreCarta(carta.valor) << " de " << carta.palo << " (" << carta.color << ")"<<endl;
    }
}

string obtenerNombreCarta(int valor) {
    switch (valor) {
        case 1: return "A"; 
        case 11: return "J";
        case 12: return "Q"; 
        case 13: return "K"; 
        default: return to_string(valor); 
    }
}

void jugar(vector<Jugador> &jugadores, vector<Carta> &mazo) {
    repartirCartas(mazo, jugadores);
    vector<Carta> mesa;
    mesa.push_back(mazo.back());
    mazo.pop_back();

    bool juegoTerminado = false;
    bool golpeado = false;
    int turnosRestantes = 0;  
    while (!juegoTerminado) {
        for (auto &jugador : jugadores) {
            if (golpeado && turnosRestantes == 0) {
                break;
            }

            cout << "Turno de " << jugador.nombre << ":"<<endl;
            cout << "Carta en la mesa: " << obtenerNombreCarta(mesa.back().valor) << " de " << mesa.back().palo << " "<<endl;
            cout << "Tu mano: "<<endl;
            mostrarMano(jugador.mano);

            cout << "Opciones: "<<endl;
            cout << "o - Tomar la carta de la mesa"<<endl;
            cout << "x - Tomar una carta del mazo"<<endl;
            if (!golpeado) {
                cout << "w - Golpear (terminar ronda)"<<endl;
            }

            char opcion;
            cin >> opcion;

            vector<Carta> manoAntesDeTomar = jugador.mano;

            if (opcion == 'o') {
                jugador.mano.push_back(mesa.back());
                mesa.pop_back();
            } else if (opcion == 'x') {
                Carta cartaTomada = mazo.back();
                mazo.pop_back();

                cout << "Tomaste la carta " << obtenerNombreCarta(cartaTomada.valor) << " de " << cartaTomada.palo << " (" << cartaTomada.color << ").\n";
                cout << "¿Quieres quedarte con esta carta? (s/n): ";
                char respuesta;
                cin >> respuesta;

                if (respuesta == 's') {
                    jugador.mano.push_back(cartaTomada);
                } else if (respuesta == 'n') {
                    mesa.push_back(cartaTomada);
                    continue; 
                }
            } else if (opcion == 'w') {
                golpeado = true;
                cout << "\n" << jugador.nombre << " ha golpeado. La ronda se termina después de esta jugada.\n";
                turnosRestantes = 1;  
                continue;
            }

            if (opcion != 'n') {
                int cartasEnMano = manoAntesDeTomar.size();
                cout << "Elige una carta para descartar (1-" << cartasEnMano << "): ";
                int indice;
                cin >> indice;

                if (indice < 1 || indice > cartasEnMano) {
                    cout << "Opción inválida, se descartará la última carta."<<endl;
                    indice = cartasEnMano;
                }

                mesa.push_back(jugador.mano[indice - 1]);
                jugador.mano.erase(jugador.mano.begin() + indice - 1);
            }

            jugador.puntaje = calcularPuntaje(jugador.mano);
            if (jugador.puntaje == 31) {
                cout << "\n" << jugador.nombre << " ha ganado con un puntaje de 31!\n";
                juegoTerminado = true;
                break;
            }
        }
        if (golpeado && turnosRestantes == 0) {
            cout << "La ronda ha sido golpeada. Ahora se verifican los puntajes finales."<<endl;
            juegoTerminado = true;
        }

        if (mazo.empty()) {
            cout << "El mazo se ha terminado. Fin del juego."<<endl;
            juegoTerminado = true;
        }

        if (turnosRestantes > 0) {
            turnosRestantes--;
        }
    }

    cout << "Puntajes finales de todos los jugadores:"<<endl;
    for (auto &jugador : jugadores) {
        cout << jugador.nombre << ": " << jugador.puntaje << " puntos"<<endl;
    }

    auto minJugador = min_element(jugadores.begin(), jugadores.end(), 
        [](const Jugador &a, const Jugador &b) {
            return a.puntaje < b.puntaje;
        });

    cout << "El jugador con menor puntaje es " << minJugador->nombre << ". ¡Ha perdido la ronda!"<<endl;
    cout << "Volviendo al menú principal..."<<endl;
}

void mostrarPuntajes(const vector<Jugador> &jugadores) {
    cout << "Puntajes de los jugadores:"<<endl;
    for (const auto &jugador : jugadores) {
        cout << jugador.nombre << ": " << jugador.puntaje << " puntos"<<endl;
    }
}

void mostrarCreditos() {
    cout << "Créditos:"<<endl;
    cout << "Este juego fue desarrollado por el Grupo 1: Pamela García y Heidy Cruz."<<endl;
}
