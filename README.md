import java.sql.*;
import java.util.*;

public class task1{
    private static final String DB_URL = "jdbc:sqlite:expenses.db";
    private static Connection connection;
    private static final Map<String, Double> exchangeRates = new HashMap<>();

    public static void main(String[] args) {
        setupDatabase();
        initializeExchangeRates();
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("1. Add Expense\n2. View Expenses\n3. Convert Currency\n4. Exit");
            int choice = scanner.nextInt();
            scanner.nextLine();
            switch (choice) {
                case 1 -> addExpense(scanner);
                case 2 -> viewExpenses();
                case 3 -> convertCurrency(scanner);
                case 4 -> { closeDatabase(); System.exit(0); }
                default -> System.out.println("Invalid choice, try again.");
            }
        }
    }

    private static void setupDatabase() {
        try {
            connection = DriverManager.getConnection(DB_URL);
            Statement stmt = connection.createStatement();
            stmt.execute("CREATE TABLE IF NOT EXISTS expenses (id INTEGER PRIMARY KEY AUTOINCREMENT, amount REAL, currency TEXT, category TEXT, date TEXT)");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void initializeExchangeRates() {
        exchangeRates.put("USD", 1.0);
        exchangeRates.put("EUR", 0.92);
        exchangeRates.put("GBP", 0.78);
        exchangeRates.put("INR", 83.0);
    }

    private static void addExpense(Scanner scanner) {
        System.out.print("Enter amount: ");
        double amount = scanner.nextDouble();
        scanner.nextLine();
        System.out.print("Enter currency (USD, EUR, GBP, INR): ");
        String currency = scanner.nextLine().toUpperCase();
        System.out.print("Enter category: ");
        String category = scanner.nextLine();
        System.out.print("Enter date (YYYY-MM-DD): ");
        String date = scanner.nextLine();
        
        try {
            PreparedStatement pstmt = connection.prepareStatement("INSERT INTO expenses (amount, currency, category, date) VALUES (?, ?, ?, ?)");
            pstmt.setDouble(1, amount);
            pstmt.setString(2, currency);
            pstmt.setString(3, category);
            pstmt.setString(4, date);
            pstmt.executeUpdate();
            System.out.println("Expense added successfully.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void viewExpenses() {
        try {
            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM expenses");
            while (rs.next()) {
                System.out.println(rs.getInt("id") + " - " + rs.getDouble("amount") + " " + rs.getString("currency") + " | " + rs.getString("category") + " | " + rs.getString("date"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void convertCurrency(Scanner scanner) {
        System.out.print("Enter amount: ");
        double amount = scanner.nextDouble();
        scanner.nextLine();
        System.out.print("Enter source currency: ");
        String from = scanner.nextLine().toUpperCase();
        System.out.print("Enter target currency: ");
        String to = scanner.nextLine().toUpperCase();

        if (!exchangeRates.containsKey(from) || !exchangeRates.containsKey(to)) {
            System.out.println("Invalid currency.");
            return;
        }

        double convertedAmount = amount * (exchangeRates.get(to) / exchangeRates.get(from));
        System.out.printf("Converted Amount: %.2f %s\n", convertedAmount, to);
    }

    private static void closeDatabase() {
        try {
            if (connection != null) connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

