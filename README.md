// Cerinta 1 - Numar nerezidenti
import java.sql.*;

public class Cerinta1 {
    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:date/bursa.db")) {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT CNP FROM Persoane");

            int count = 0;
            while (rs.next()) {
                String cnp = rs.getString("CNP");
                if (cnp.startsWith("8") || cnp.startsWith("9")) {
                    count++;
                }
            }
            System.out.println("CERINTA 1: Numar nerezidenti " + count);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

// Cerinta 2 - Numar tranzactii per simbol
import java.io.*;
import java.util.*;

public class Cerinta2 {
    public static void main(String[] args) {
        Map<String, Integer> tranzactii = new HashMap<>();

        try (BufferedReader br = new BufferedReader(new FileReader("date/bursa_tranzactii.txt"))) {
            String linie;
            while ((linie = br.readLine()) != null) {
                String[] valori = linie.split(",");
                String simbol = valori[1].trim();
                tranzactii.put(simbol, tranzactii.getOrDefault(simbol, 0) + 1);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        System.out.println("CERINTA 2: Numar tranzactii:");
        for (Map.Entry<String, Integer> entry : tranzactii.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue() + " tranzactii");
        }
    }
}

// Cerinta 3 - Scriere simboluri unice
import java.io.*;
import java.util.*;

public class Cerinta3 {
    public static void main(String[] args) {
        Set<String> simboluri = new TreeSet<>();

        try (BufferedReader br = new BufferedReader(new FileReader("date/bursa_tranzactii.txt"))) {
            String linie;
            while ((linie = br.readLine()) != null) {
                String[] valori = linie.split(",");
                simboluri.add(valori[1].trim());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        try (BufferedWriter bw = new BufferedWriter(new FileWriter("date/simboluri.txt"))) {
            for (String simbol : simboluri) {
                bw.write(simbol);
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// Cerinta 4 - Portofolii clienti
import java.io.*;
import java.sql.*;
import java.util.*;

public class Cerinta4 {
    public static void main(String[] args) {
        Map<Integer, String> clienti = new HashMap<>();

        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:date/bursa.db")) {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT Cod, Nume FROM Persoane");

            while (rs.next()) {
                clienti.put(rs.getInt("Cod"), rs.getString("Nume"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        Map<Integer, Map<String, Integer>> portofolii = new TreeMap<>();

        try (BufferedReader br = new BufferedReader(new FileReader("date/bursa_tranzactii.txt"))) {
            String linie;
            while ((linie = br.readLine()) != null) {
                String[] v = linie.split(",");
                int cod = Integer.parseInt(v[0].trim());
                String simbol = v[1].trim();
                String tip = v[2].trim();
                int cantitate = Integer.parseInt(v[3].trim());

                portofolii.putIfAbsent(cod, new TreeMap<>());
                Map<String, Integer> actiuni = portofolii.get(cod);

                int factor = tip.equalsIgnoreCase("cumparare") ? 1 : -1;
                actiuni.put(simbol, actiuni.getOrDefault(simbol, 0) + factor * cantitate);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        System.out.println("CERINTA 4: Portofolii clienti");
        for (Map.Entry<Integer, Map<String, Integer>> entry : portofolii.entrySet()) {
            String nume = clienti.getOrDefault(entry.getKey(), "Necunoscut");
            System.out.println("  " + nume);
            for (Map.Entry<String, Integer> actiune : entry.getValue().entrySet()) {
                System.out.println("    " + actiune.getKey() + " - " + actiune.getValue());
            }
        }
    }
}

// Cerinta 5 - Server TCP
import java.io.*;
import java.net.*;
import java.util.*;

public class Server {
    private static Map<String, Integer> simbolTranzactii = new HashMap<>();

    public static void main(String[] args) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("date/bursa_tranzactii.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                String simbol = line.split(",")[1].trim();
                simbolTranzactii.put(simbol, simbolTranzactii.getOrDefault(simbol, 0) + 1);
            }
        }

        ServerSocket serverSocket = new ServerSocket(1234);
        System.out.println("Server pornit...");

        while (true) {
            Socket client = serverSocket.accept();
            new Thread(() -> {
                try {
                    BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
                    PrintWriter out = new PrintWriter(client.getOutputStream(), true);

                    String simbol = in.readLine();
                    System.out.println("Server a primit simbolul: " + simbol);
                    int tranzactii = simbolTranzactii.getOrDefault(simbol, 0);
                    out.println("Numar tranzactii pentru " + simbol + ": " + tranzactii);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}

// Cerinta 5 - Client TCP
import java.io.*;
import java.net.*;

public class Client implements Runnable {
    private String simbol;

    public Client(String simbol) {
        this.simbol = simbol;
    }

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

    public static void main(String[] args) {
        new Thread(new Client("SNP")).start();
        new Thread(new Client("BRD")).start();
    }
}

