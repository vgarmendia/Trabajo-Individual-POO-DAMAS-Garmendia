public class App {
    public static void main(String[] args) {
        Game game = Game.getInstance();
        game.startGame();
    }
}


public class Casilla {
    private final int fila;
    private final int columna;
    private Ficha ficha;

    public Casilla(int fila, int columna, Ficha ficha) {
        this.fila = fila;
        this.columna = columna;
        this.ficha = ficha;
    }

    public int getFila() {
        return fila;
    }

    public int getColumna() {
        return columna;
    }

    public Ficha getFicha() {
        return ficha;
    }

    public void setFicha(Ficha ficha) {
        this.ficha = ficha;
    }

    public void voltearFicha() {
        if (ficha != null) {
            ficha.voltearFicha();
        }
    }

    public boolean isVolteada() {
        return ficha != null && ficha.getEstado();
    }
}


public class Ficha {
    private String codigo;
    private boolean estado;

    public Ficha(String codigo) {
        this.codigo = codigo;
        this.estado = false;
    }

    public String getCodigo() {
        return codigo;
    }

    public boolean getEstado() {
        return estado;
    }

    public void setEstado(boolean estado) {
        this.estado = estado;
    }

    public void voltearFicha() {
        this.estado = !this.estado;
    }
}


public class FichaHandler {
    public void voltearFicha(Casilla casilla) {
        casilla.voltearFicha();
    }

    public boolean compararFichas(Casilla casilla1, Casilla casilla2) {
        Ficha ficha1 = casilla1.getFicha();
        Ficha ficha2 = casilla2.getFicha();
        return ficha1 != null && ficha2 != null && ficha1.getCodigo().equals(ficha2.getCodigo());
    }
}

import java.util.Scanner;

public class Game {
    private static Game instance;
    private final Tablero tablero;
    private final Jugador jugador1;
    private final Jugador jugador2;
    private Casilla casilla1;
    private Casilla casilla2;

    private final FichaHandler fichaHandler;
    private final JugadorHandler jugadorHandler;
    private final TableroHandler tableroHandler;
    private final Scanner scanner;

    private Game() {
        this.tablero = new Tablero();
        this.jugador1 = new Jugador(1);
        this.jugador2 = new Jugador(2);
        this.casilla1 = null;
        this.casilla2 = null;

        this.fichaHandler = new FichaHandler();
        this.jugadorHandler = new JugadorHandler();
        this.tableroHandler = new TableroHandler();
        this.scanner = new Scanner(System.in);
    }

    public static Game getInstance() {
        if (instance == null) {
            instance = new Game();
        }
        return instance;
    }

    public void startGame() {
        inicializarJuego();

        while (true) {
            System.out.print("Ingrese fila y columna: ");
            int fila = scanner.nextInt();
            int columna = scanner.nextInt();
            jugar(fila, columna);
        }
    }

    public void inicializarJuego() {
        tableroHandler.inicializarTablero(tablero);
        jugador1.setTurno(true);
        jugador2.setTurno(false);
        mostrarTurno(jugador1);
        inicializarFichaPorTurno();
        tableroHandler.mostrarTablero(tablero);
    }

    public void jugar(int fila, int columna) {
        if (!verificarFueElegida(casilla1)) {
            casilla1 = elegirCasilla(fila, columna);
        } else {
            casilla2 = elegirCasilla(fila, columna);
            if (fichaHandler.compararFichas(casilla1, casilla2)) {
                actualizarPuntuacion(compararTurnos(jugador1, jugador2));
                if (verificarFinDeJuego(tablero)) {
                    finalizarJuego();
                }
            } else {
                alternarTurno();
                tableroHandler.mostrarTablero(tablero);
            }
            inicializarFichaPorTurno();
            mostrarTurno(compararTurnos(jugador1, jugador2));
        }
    }

    private Casilla elegirCasilla(int fila, int columna) {
        Casilla casilla = tableroHandler.getCasilla(tablero, fila, columna);
        fichaHandler.voltearFicha(casilla);
        tableroHandler.mostrarTablero(tablero);
        return casilla;
    }

    private void inicializarFichaPorTurno() {
        casilla1 = null;
        casilla2 = null;
    }

    private void alternarTurno() {
        jugador1.cambiarTurno();
        jugador2.cambiarTurno();
        casilla1.voltearFicha();
        casilla2.voltearFicha();
    }

    private void actualizarPuntuacion(Jugador jugador) {
        jugadorHandler.incrementarPuntaje(jugador);
        jugadorHandler.mostrarPuntaje(jugador);
    }

    private boolean verificarFinDeJuego(Tablero tablero) {
        return tableroHandler.contarFichasVolteadas(tablero) == 24;
    }

    private void finalizarJuego() {
        System.out.println("Fin del juego");
        if (verificarEmpate(jugador1, jugador2)) {
            System.out.println("No hay un ganador, Empate de " + jugador1.getPuntaje() + " puntos");
        } else {
            Jugador ganador = verificarGanador(jugador1, jugador2);
            System.out.println("Ganador: Jugador " + ganador.getNumero() + " con " + ganador.getPuntaje() + " puntos");
        }
    }

    private boolean verificarEmpate(Jugador jugador1, Jugador jugador2) {
        return jugador1.getPuntaje() == jugador2.getPuntaje();
    }

    private Jugador verificarGanador(Jugador jugador1, Jugador jugador2) {
        return jugador1.getPuntaje() > jugador2.getPuntaje() ? jugador1 : jugador2;
    }

    private Jugador compararTurnos(Jugador jugador1, Jugador jugador2) {
        return jugador1.getTurno() ? jugador1 : jugador2;
    }

    private void mostrarTurno(Jugador jugador) {
        System.out.println("Turno de Jugador " + jugador.getNumero());
    }

    private boolean verificarFueElegida(Casilla casilla) {
        return casilla != null;
    }
}



public class Jugador {
    private final int numero;
    private int puntaje;
    private boolean turno;

    public Jugador(int numero) {
        this.numero = numero;
        this.puntaje = 0;
        this.turno = false;
    }

    public void incrementarPuntaje() {
        this.puntaje++;
    }

    public void mostrarPuntaje() {
        System.out.println("Jugador " + numero + " tiene " + puntaje + " puntos.");
    }

    public void cambiarTurno() {
        this.turno = !this.turno;
    }

    public int getNumero() {
        return numero;
    }

    public int getPuntaje() {
        return puntaje;
    }

    public boolean getTurno() {
        return turno;
    }

    public void setTurno(boolean turno) {
        this.turno = turno;
    }
}


public class JugadorHandler {
    public void incrementarPuntaje(Jugador jugador) {
        jugador.incrementarPuntaje();
    }

    public void mostrarPuntaje(Jugador jugador) {
        jugador.mostrarPuntaje();
    }

    public void cambiarTurno(Jugador jugador) {
        jugador.cambiarTurno();
    }
}


public class Tablero {
    private static final String[] CODIGOS = {"A", "B", "C", "D", "E", "F", "G", "H"};
    private final Casilla[][] matriz;

    public Tablero() {
        this.matriz = new Casilla[8][8];
    }

    public void inicializarTablero() {
        // Implementación para inicializar el tablero y colocar las fichas
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {
                String codigo = CODIGOS[(i * 8 + j) % CODIGOS.length];
                matriz[i][j] = new Casilla(i, j, new Ficha(codigo));
            }
        }
    }

    public void mostrarTablero() {
        // Implementación para mostrar el tablero en consola
        for (Casilla[] fila : matriz) {
            for (Casilla casilla : fila) {
                if (casilla.isVolteada()) {
                    System.out.print(casilla.getFicha().getCodigo() + " ");
                } else {
                    System.out.print("X ");
                }
            }
            System.out.println();
        }
    }

    public int contarFichasVolteadas() {
        int contador = 0;
        for (Casilla[] fila : matriz) {
            for (Casilla casilla : fila) {
                if (casilla.isVolteada()) {
                    contador++;
                }
            }
        }
        return contador;
    }

    public Casilla getCasilla(int fila, int columna) {
        return matriz[fila][columna];
    }
}


public class TableroHandler {
    public void inicializarTablero(Tablero tablero) {
        tablero.inicializarTablero();
    }

    public void mostrarTablero(Tablero tablero) {
        tablero.mostrarTablero();
    }

    public int contarFichasVolteadas(Tablero tablero) {
        return tablero.contarFichasVolteadas();
    }

    public Casilla getCasilla(Tablero tablero, int fila, int columna) {
        return tablero.getCasilla(fila, columna);
    }
}
