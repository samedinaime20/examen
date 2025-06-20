// Server.java
import java.io.*;
import java.net.*;
import java.util.*;

public class Server {
    private static Map<String, Integer> tranzactii = new HashMap<>();

    public static void main(String[] args) throws IOException {
        // Incarca tranzactii in memorie
        try (BufferedReader br = new BufferedReader(new FileReader("date/bursa_tranzactii.txt"))) {
            String linie;
            while ((linie = br.readLine()) != null) {
                String simbol = linie.split(",")[1].trim();
                tranzactii.put(simbol, tranzactii.getOrDefault(simbol, 0) + 1);
            }
        }

        ServerSocket serverSocket = new ServerSocket(1234);
        System.out.println("Server pornit pe portul 1234...");

        while (true) {
            Socket socket = serverSocket.accept();
            new Thread(() -> {
                try {
                    BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

                    String simbol = in.readLine();
                    System.out.println("Server a primit simbolul: " + simbol);
                    int rezultat = tranzactii.getOrDefault(simbol, 0);
                    out.println("Numar tranzactii pentru " + simbol + ": " + rezultat);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}


// Client.java
import java.io.*;
import java.net.*;

public class Client implements Runnable {
    private final String simbol;

    public Client(String simbol) {
        this.simbol = simbol;
    }

    @Override
    public void run() {
        try (Socket socket = new Socket("localhost", 1234);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {

            out.println(simbol);
            String raspuns = in.readLine();
            System.out.println("Client a primit: " + raspuns);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // Program principal - lansează server + 2 clienți
    public static void main(String[] args) {
        // Pornim serverul într-un fir separat
        new Thread(() -> {
            try {
                Server.main(null);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();

        // Așteptăm puțin pentru ca serverul să pornească
        try { Thread.sleep(1500); } catch (InterruptedException e) { }

        // Pornim 2 clienți
        new Thread(new Client("SNP")).start();
        new Thread(new Client("BRD")).start();
    }
}

