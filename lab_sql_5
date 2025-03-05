Лабораторная работа по УД номер 5

Предметная область - книги, имеющие такие харрактеристики, как id, название, автора и доступность

Код реализует создание ролей и прав для БД, создание самой БД с таблицей и нужными хранимыми процедурами, а также GUI для управления  

Для запуска требуется наличие такой БД, как postgres и пароль для подключения к ней 




Сама реализация: 

```
import javax.swing.*;
import java.awt.*;
import java.sql.*;
import java.util.Vector;

public class DatabaseManager extends JFrame {
    private String role;
    private Connection adminConn; // соединение к базе postgres (для административных операций таких как, создание БД booksDB, создание необходимых процедур и ролей)
    private Connection conn;      // соединение к базе bookdb (для операций с самой БД)

    public DatabaseManager(String role) {
        super("Book Manager - " + role);
        this.role = role;
        initUI();

        // Пробуем установить соединение с базой bookdb
        // Если базы нет, пользователю (администратору) предстоит её создать
        try {
            conn = getConnection("bookdb");
        } catch (SQLException ex) {
            conn = null;
        }

        // Для администратора создаём дополнительное соединение к базе postgres
        if (role.equals("admin")) {
            try {
                String url = "jdbc:postgresql://localhost:5432/postgres";
                String user = "postgres";
                String password = "Kirillica2005";
                Connection adminConn = DriverManager.getConnection(url, user, password);
            } catch (SQLException ex) {
                adminConn = null;
                showError("Ошибка подключения к базе postgres: " + ex.getMessage());
            }
        }
    }

    // Метод для получения соединения по имени базы данных
    private Connection getConnection(String dbName) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/" + dbName;
        // Используем учётные данные, соответствующие выбранному режиму доступа
        String user = role.equals("admin") ? "admin" : "guest";
        String password = role.equals("admin") ? "admin" : "guest";
        return DriverManager.getConnection(url, user, password);
    }

    // Инициализация графического интерфейса
    private void initUI() {
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setSize(600, 400);
        setLocationRelativeTo(null);

        JPanel panel = new JPanel(new GridLayout(0, 1, 5, 5));

        if (role.equals("admin")) {
            JButton btnCreateDB = new JButton("Создать базу данных и процедуры");
            btnCreateDB.addActionListener(e -> createDatabase());
            panel.add(btnCreateDB);

            JButton btnDropDB = new JButton("Удалить базу данных");
            btnDropDB.addActionListener(e -> dropDatabase());
            panel.add(btnDropDB);

            JButton btnCreateTable = new JButton("Создать таблицу");
            btnCreateTable.addActionListener(e -> createTable());
            panel.add(btnCreateTable);

            JButton btnClearTable = new JButton("Очистить таблицу");
            btnClearTable.addActionListener(e -> clearTable());
            panel.add(btnClearTable);

            JButton btnInsert = new JButton("Добавить книгу");
            btnInsert.addActionListener(e -> insertBook());
            panel.add(btnInsert);

            JButton btnUpdate = new JButton("Обновить книгу");
            btnUpdate.addActionListener(e -> updateBook());
            panel.add(btnUpdate);

            JButton btnDelete = new JButton("Удалить книгу по названию");
            btnDelete.addActionListener(e -> deleteBookByTitle());
            panel.add(btnDelete);
        }

        // Доступно для обоих режимов (просмотр и поиск)
        JButton btnShowAll = new JButton("Показать все книги");
        btnShowAll.addActionListener(e -> getAllBooks());
        panel.add(btnShowAll);

        JButton btnSearch = new JButton("Поиск по названию");
        btnSearch.addActionListener(e -> searchByTitle());
        panel.add(btnSearch);

        add(panel, BorderLayout.CENTER);
    }

    // Метод, создающий базу данных, роли и все необходимые процедуры
    private void createDatabase() {
        if (!role.equals("admin")) {
            showError("У вас недостаточно прав для создания базы данных.");
            return;
        }

        // Подключаемся к базе postgres для создания ролей и базы данных
        try (Connection adminConn = DriverManager.getConnection(
                "jdbc:postgresql://localhost:5432/postgres",
                "postgres",
            "Kirillica2005");
             Statement stmt = adminConn.createStatement()) {

            // Создаем роль admin, если она не существует
            String sqlCreateAdmin = "DO $$ " +
                    "BEGIN " +
                    "   IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'admin') THEN " +
                    "       CREATE ROLE admin LOGIN PASSWORD 'admin' SUPERUSER; " +
                    "   END IF; " +
                    "END $$;";
            stmt.execute(sqlCreateAdmin);

            // Создаем роль guest, если она не существует
            String sqlCreateGuest = "DO $$ " +
                    "BEGIN " +
                    "   IF NOT EXISTS (SELECT 1 FROM pg_roles WHERE rolname = 'guest') THEN " +
                    "       CREATE ROLE guest LOGIN PASSWORD 'guest'; " +
                    "   END IF; " +
                    "END $$;";
            stmt.execute(sqlCreateGuest);



            // Проверяем наличие базы данных bookdb и создаём её, если отсутствует
            String sqlCheckDB = "SELECT 1 FROM pg_database WHERE datname = 'bookdb'";
            try (ResultSet rs = stmt.executeQuery(sqlCheckDB)) {
                if (!rs.next()) {
                    stmt.execute("CREATE DATABASE bookdb");
                }
            }

            JOptionPane.showMessageDialog(this, "База данных и роли созданы (если не существовали).");

        } catch (SQLException ex) {
            showError("Ошибка при создании базы данных и ролей: " + ex.getMessage());
            return;
        }

        // Подключаемся к базе bookdb для назначения прав
        try (Connection bookConn = DriverManager.getConnection(
                "jdbc:postgresql://localhost:5432/bookdb",
                "postgres",
            "Kirillica2005");
             Statement stmt = bookConn.createStatement()) {

            // Назначаем права доступа
            String grantPermissions = """
            GRANT CONNECT ON DATABASE bookdb TO admin, guest;
            GRANT USAGE ON SCHEMA public TO admin, guest;
            GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin;
            GRANT SELECT ON ALL TABLES IN SCHEMA public TO guest;
            GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO admin;
            GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO guest;
        """;

            stmt.execute(grantPermissions);

            JOptionPane.showMessageDialog(this, "Права доступа назначены.");

        } catch (SQLException ex) {
            showError("Ошибка при назначении прав доступа: " + ex.getMessage());
        }

        // Подключаемся к базе bookdb для создания хранимых процедур
        try (Connection bookConn = DriverManager.getConnection(
                "jdbc:postgresql://localhost:5432/bookdb",
                "admin",
            "admin");
             Statement stmt = bookConn.createStatement()) {

            // Процедура для создания таблицы books
            String sqlProcCreateTable = "CREATE OR REPLACE PROCEDURE sp_create_table() " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   IF NOT EXISTS (SELECT 1 FROM information_schema.tables WHERE table_name = 'books') THEN " +
                    "       EXECUTE 'CREATE TABLE books (" +
                    "                   id SERIAL PRIMARY KEY, " +
                    "                   title VARCHAR(255), " +
                    "                   author VARCHAR(255), " +
                    "                   available BOOLEAN)'; " +
                    "   END IF; " +
                    "END; $$;";
            stmt.execute(sqlProcCreateTable);

            // Процедура для очистки таблицы books
            String sqlProcClearTable = "CREATE OR REPLACE PROCEDURE sp_clear_table() " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   EXECUTE 'TRUNCATE TABLE books'; " +
                    "END; $$;";
            stmt.execute(sqlProcClearTable);

            // Процедура для добавления книги
            String sqlProcInsertBook = "CREATE OR REPLACE PROCEDURE sp_insert_book(" +
                    "p_title VARCHAR, p_author VARCHAR, p_available BOOLEAN) " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   INSERT INTO books(title, author, available) VALUES (p_title, p_author, p_available); " +
                    "END; $$;";
            stmt.execute(sqlProcInsertBook);

            // Процедура для поиска по названию (частичное совпадение)
            String sqlProcSearchByTitle = "CREATE OR REPLACE PROCEDURE sp_search_by_title(" +
                    "p_title VARCHAR, OUT p_cursor refcursor) " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   OPEN p_cursor FOR " +
                    "       SELECT * FROM books WHERE title ILIKE '%' || p_title || '%'; " +
                    "END; $$;";
            stmt.execute(sqlProcSearchByTitle);

            // Процедура для получения всех записей
            String sqlProcGetAllBooks = "CREATE OR REPLACE PROCEDURE sp_get_all_books(OUT p_cursor refcursor) " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "    OPEN p_cursor FOR SELECT * FROM books; " +
                    "END; $$;";
            stmt.execute(sqlProcGetAllBooks);

            // Процедура для обновления записи по id
            String sqlProcUpdateBook = "CREATE OR REPLACE PROCEDURE sp_update_book(" +
                    "p_id INT, p_title VARCHAR, p_author VARCHAR, p_available BOOLEAN) " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   UPDATE books SET title = p_title, author = p_author, available = p_available WHERE id = p_id; " +
                    "END; $$;";
            stmt.execute(sqlProcUpdateBook);

            // Процедура для удаления книги по названию
            String sqlProcDeleteByTitle = "CREATE OR REPLACE PROCEDURE sp_delete_by_title(" +
                    "p_title VARCHAR) " +
                    "LANGUAGE plpgsql AS $$ " +
                    "BEGIN " +
                    "   DELETE FROM books WHERE title = p_title; " +
                    "END; $$;";
            stmt.execute(sqlProcDeleteByTitle);

            // Выдача прав на выполнение процедур
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_create_table() TO admin, guest;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_clear_table() TO admin, guest;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_insert_book(VARCHAR, VARCHAR, BOOLEAN) TO admin;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_update_book(INT, VARCHAR, VARCHAR, BOOLEAN) TO admin;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_delete_by_title(VARCHAR) TO admin;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_get_all_books() TO admin, guest;");
            stmt.execute("GRANT EXECUTE ON PROCEDURE sp_search_by_title(VARCHAR, OUT refcursor) TO admin, guest;");

            JOptionPane.showMessageDialog(this, "Все необходимые процедуры созданы в базе bookdb.");

            // Обновляем соединение для последующих операций
            conn = getConnection("bookdb");
        } catch (SQLException ex) {
            showError("Ошибка при создании хранимых процедур: " + ex.getMessage());
        }
    }

    // Удаление базы данных bookdb (только для администратора)
    private void dropDatabase() {
        if (!role.equals("admin")) {
            showError("У вас недостаточно прав для удаления базы данных.");
            return;
        }
        try (Connection postgresConn = DriverManager.getConnection(
                "jdbc:postgresql://localhost:5432/postgres",
                "admin",
                "admin");
             Statement stmt = postgresConn.createStatement()) {

            // Завершаем все подключения к bookdb
            String sqlTerminate = "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'bookdb';";
            stmt.execute(sqlTerminate);
            stmt.execute("DROP DATABASE IF EXISTS bookdb");
            JOptionPane.showMessageDialog(this, "База данных bookdb удалена.");
            conn = null;
        } catch (SQLException ex) {
            showError("Ошибка при удалении базы данных: " + ex.getMessage());
        }
    }

    // Вызов процедуры sp_create_table
    private void createTable() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        try (CallableStatement stmt = conn.prepareCall("CALL sp_create_table()")) {
            stmt.execute();
            JOptionPane.showMessageDialog(this, "Таблица books создана (если ранее не существовала).");
        } catch (SQLException ex) {
            showError("Ошибка при создании таблицы: " + ex.getMessage());
        }
    }

    // Вызов процедуры sp_clear_table
    private void clearTable() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        try (CallableStatement stmt = conn.prepareCall("CALL sp_clear_table()")) {
            stmt.execute();
            JOptionPane.showMessageDialog(this, "Таблица books очищена.");
        } catch (SQLException ex) {
            showError("Ошибка при очистке таблицы: " + ex.getMessage());
        }
    }

    // Вызов процедуры sp_insert_book
    private void insertBook() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        JTextField titleField = new JTextField();
        JTextField authorField = new JTextField();
        JCheckBox availableBox = new JCheckBox("Доступна");
        Object[] fields = {
                "Название:", titleField,
                "Автор:", authorField,
                "Доступна:", availableBox
        };
        int option = JOptionPane.showConfirmDialog(this, fields, "Добавить книгу", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            String title = titleField.getText();
            String author = authorField.getText();
            boolean available = availableBox.isSelected();
            try (CallableStatement stmt = conn.prepareCall("CALL sp_insert_book(?, ?, ?)")) {
                stmt.setString(1, title);
                stmt.setString(2, author);
                stmt.setBoolean(3, available);
                stmt.execute();
                JOptionPane.showMessageDialog(this, "Книга добавлена.");
            } catch (SQLException ex) {
                showError("Ошибка при добавлении книги: " + ex.getMessage());
            }
        }
    }

    // Вызов процедуры sp_update_book
    private void updateBook() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        JTextField idField = new JTextField();
        JTextField titleField = new JTextField();
        JTextField authorField = new JTextField();
        JCheckBox availableBox = new JCheckBox("Доступна");
        Object[] fields = {
                "ID книги:", idField,
                "Новое название:", titleField,
                "Новый автор:", authorField,
                "Доступна:", availableBox
        };
        int option = JOptionPane.showConfirmDialog(this, fields, "Обновить книгу", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            try {
                int id = Integer.parseInt(idField.getText());
                String title = titleField.getText();
                String author = authorField.getText();
                boolean available = availableBox.isSelected();
                try (CallableStatement stmt = conn.prepareCall("CALL sp_update_book(?, ?, ?, ?)")) {
                    stmt.setInt(1, id);
                    stmt.setString(2, title);
                    stmt.setString(3, author);
                    stmt.setBoolean(4, available);
                    stmt.execute();
                    JOptionPane.showMessageDialog(this, "Книга обновлена.");
                }
            } catch (NumberFormatException ex) {
                showError("Неверный формат ID.");
            } catch (SQLException ex) {
                showError("Ошибка при обновлении книги: " + ex.getMessage());
            }
        }
    }

    // Вызов процедуры sp_delete_by_title
    private void deleteBookByTitle() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        String title = JOptionPane.showInputDialog(this, "Введите название книги для удаления:");
        if (title != null && !title.trim().isEmpty()) {
            try (CallableStatement stmt = conn.prepareCall("CALL sp_delete_by_title(?)")) {
                stmt.setString(1, title);
                stmt.execute();
                JOptionPane.showMessageDialog(this, "Книга(и) с названием '" + title + "' удалена(ы).");
            } catch (SQLException ex) {
                showError("Ошибка при удалении книги: " + ex.getMessage());
            }
        }
    }

    // Вызов процедуры sp_get_all_books и отображение результата
    private void getAllBooks() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }

        try {
            conn.setAutoCommit(false); // ВАЖНО: Транзакция для работы с курсором

            try (CallableStatement stmt = conn.prepareCall("CALL sp_get_all_books(?)")) {
                stmt.registerOutParameter(1, Types.OTHER);
                stmt.execute();

                try (ResultSet rs = (ResultSet) stmt.getObject(1)) {
                    displayResultSet(rs);
                }
            }

            conn.commit(); // Закрываем транзакцию
        } catch (SQLException ex) {
            showError("Ошибка при получении списка книг: " + ex.getMessage());
            try {
                conn.rollback(); // Откат в случае ошибки
            } catch (SQLException rollbackEx) {
                showError("Ошибка при откате транзакции: " + rollbackEx.getMessage());
            }
        } finally {
            try {
                conn.setAutoCommit(true); // Восстанавливаем автокоммит
            } catch (SQLException ex) {
                showError("Ошибка при восстановлении автокоммита: " + ex.getMessage());
            }
        }
    }

    // Вызов процедуры sp_search_by_title и отображение результата
    private void searchByTitle() {
        if (conn == null) {
            showError("Нет подключения к базе данных bookdb");
            return;
        }
        String title = JOptionPane.showInputDialog(this, "Введите название для поиска:");
        if (title != null && !title.trim().isEmpty()) {
            try {
                conn.setAutoCommit(false); // Включаем транзакцию для refcursor

                try (CallableStatement stmt = conn.prepareCall("CALL sp_search_by_title(?, ?)")) {
                    stmt.setString(1, title);
                    stmt.registerOutParameter(2, Types.OTHER);
                    stmt.execute();
                    ResultSet rs = (ResultSet) stmt.getObject(2);
                    displayResultSet(rs);
                    rs.close();
                } catch (SQLException ex) {
                    showError("Ошибка при поиске книги: " + ex.getMessage());
                }

                conn.commit(); // Завершаем транзакцию
            } catch (SQLException ex) {
                showError("Ошибка при поиске книги: " + ex.getMessage());
            }
        }
    }

    // Отображение ResultSet в виде таблицы
    private void displayResultSet(ResultSet rs) throws SQLException {
        ResultSetMetaData meta = rs.getMetaData();
        int columnCount = meta.getColumnCount();
        Vector<String> columnNames = new Vector<>();
        for (int i = 1; i <= columnCount; i++) {
            columnNames.add(meta.getColumnName(i));
        }
        Vector<Vector<Object>> data = new Vector<>();
        while (rs.next()) {
            Vector<Object> row = new Vector<>();
            for (int i = 1; i <= columnCount; i++) {
                row.add(rs.getObject(i));
            }
            data.add(row);
        }
        JTable table = new JTable(data, columnNames);
        JScrollPane scrollPane = new JScrollPane(table);
        table.setFillsViewportHeight(true);
        JFrame frame = new JFrame("Результаты");
        frame.setSize(500, 300);
        frame.add(scrollPane);
        frame.setLocationRelativeTo(this);
        frame.setVisible(true);
    }

    // Метод для вывода сообщений об ошибках
    private void showError(String message) {
        JOptionPane.showMessageDialog(this, message, "Ошибка", JOptionPane.ERROR_MESSAGE);
    }

    // Точка входа в приложение
    public static void main(String[] args) {
        try {
            Class.forName("org.postgresql.Driver");
        } catch (ClassNotFoundException ex) {
            JOptionPane.showMessageDialog(null, "PostgreSQL JDBC Driver не найден.");
            return;
        }
        String[] options = {"Администратор", "Гость"};
        int selection = JOptionPane.showOptionDialog(null, "Выберите режим доступа:",
                "Выбор режима", JOptionPane.DEFAULT_OPTION, JOptionPane.INFORMATION_MESSAGE,
                null, options, options[0]);
        if (selection == -1) return;
        String role = (selection == 0) ? "admin" : "guest";
        SwingUtilities.invokeLater(() -> {
            DatabaseManager app = new DatabaseManager(role);
            app.setVisible(true);
        });
    }
}
```

