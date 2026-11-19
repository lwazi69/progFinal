# progFinal

Main class

mport java.util.Scanner;

/**
 *
 * @author Student
 */
public class PoePart3 {

    /**
     * @param args the command line arguments
     */
      public static void main(String[] args) {

        Scanner scan = new Scanner(System.in);

        Login account = new Login();

        System.out.println("========== REGISTRATION ==========");

        System.out.print("Enter username: ");
        String username = scan.nextLine();

        System.out.print("Enter password: ");
        String password = scan.nextLine();

        System.out.print("Enter cellphone number: ");
        String cellphone = scan.nextLine();

        System.out.println(account.registerUser(
                username, password, cellphone));

        if (!account.registerStatus(username, password, cellphone)
                || !username.equals(account.userName)
                || !password.equals(account.passWord)
                || !cellphone.equals(account.cellPhone)) {

            System.out.println("Registration failed. Closing program.");

            scan.close();
            return;
        }

        System.out.println("\n========== LOGIN ==========");

        System.out.print("Enter username: ");
        String loginName = scan.nextLine();

        System.out.print("Enter password: ");
        String loginPassword = scan.nextLine();

        boolean loginStatus = account.loginUser(
                loginName, loginPassword);

        System.out.println(
                account.returnLoginStatus(loginStatus));

        if (!loginStatus) {
            scan.close();
            return;
        }

        MessageManager manager = new MessageManager(scan);

        int choice;

        do {

            System.out.println("\n========== MAIN MENU ==========");
            System.out.println("1. Send Messages");
            System.out.println("2. View Sent Messages");
            System.out.println("3. Quit");
            System.out.println("4. Stored Messages");

            System.out.print("Choose an option: ");

            try {
                choice = Integer.parseInt(scan.nextLine());

            } catch (NumberFormatException e) {
                choice = -1;
            }

            switch (choice) {

                case 1:
                    manager.sendingMessages();
                    break;

                case 2:
                    manager.printMessages();

                    System.out.println(
                            "Total messages sent: "
                            + manager.returnTotalMessages());

                    break;

                case 3:
                    System.out.println("Thank you for using the application.");
                    break;

                case 4:
                    manager.storedMessagesMenu();
                    break;

                default:
                    System.out.println("Invalid option.");
            }

        } while (choice != 3);

        scan.close();
    }
}

Registration code

public class Regestration {

    String userName;
    String passWord;
    String cellPhone;

    public boolean checkUserName(String username) {
        return username != null
                && username.contains("_")
                && username.length() <= 5;
    }

    public boolean checkPasswordComplexity(String password) {
        return password != null
                && password.length() >= 8
                && password.matches(".*[A-Z].*")
                && password.matches(".*[a-z].*")
                && password.matches(".*[!@#$%^&*].*");
    }

    public boolean checkCellPhoneNumber(String cellphone) {
        return cellphone != null
                && (cellphone.matches("^\\+27[0-9]{9}$")
                || cellphone.matches("^0[0-9]{9}$"));
    }

    public String registerUser(String username, String password,
                               String cellphone) {

        if (checkUserName(username)
                && checkPasswordComplexity(password)
                && checkCellPhoneNumber(cellphone)) {

            this.userName = username;
            this.passWord = password;
            this.cellPhone = cellphone;

            return "Registration was a success";

        } else {
            return "Registration wasnt successful";
        }
    }

    public boolean registerStatus(String username, String password,
                                  String cellphone) {

        return checkUserName(username)
                && checkPasswordComplexity(password)
                && checkCellPhoneNumber(cellphone);
    }
}

login code

public class Login extends Regestration {

    public boolean loginUser(String name, String password) {

        return name != null
                && password != null
                && name.equals(userName)
                && password.equals(passWord);
    }

    public String returnLoginStatus(boolean loginStatus) {

        if (loginStatus) {
            return "Login successful!";
        } else {
            return "Login failed. Incorrect username or password.";
        }
    }
}

messageManager

import java.util.ArrayList;
import java.util.HashSet;
import java.util.Scanner;
import java.util.Random;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.Map;
import java.util.LinkedHashMap;
import java.util.concurrent.ThreadLocalRandom;

public class MessageManager {

    ArrayList<String> sentMessages = new ArrayList<>();
    ArrayList<String> messageIdsList = new ArrayList<>();
    HashSet<String> messageIds = new HashSet<>();
    ArrayList<String> msgHash = new ArrayList<>();
    ArrayList<String> recipientNumber = new ArrayList<>();
    ArrayList<String> messageStatus = new ArrayList<>();
    ArrayList<String> disregardedMessages = new ArrayList<>();
    ArrayList<String> senders = new ArrayList<>();

    ArrayList<StoredMessage> storedMessages = new ArrayList<>();

    Scanner input;
    Random random = new Random();

    int numMessageSent = 0;
    int nextMessageNumber = 1;

    File jsonFile = new File("messages.json");

    public MessageManager(Scanner input) {
        this.input = input;
        loadStoredMessages();
    }

    public static class StoredMessage {

        String messageID;
        int messageNumber;
        String sender;
        String recipient;
        String message;
        String messageHash;
        String status;

        StoredMessage(String id, int number, String sender,
                      String recipient, String message,
                      String hash, String status) {

            this.messageID = id;
            this.messageNumber = number;
            this.sender = sender;
            this.recipient = recipient;
            this.message = message;
            this.messageHash = hash;
            this.status = status;
        }
    }

    public boolean checkMessageID(String id) {
        return id != null && id.matches("[0-9]{1,10}");
    }

    public String generateMessageID() {

        String id;

        do {
            id = String.valueOf(
                    ThreadLocalRandom.current().nextLong(
                            1000000000L, 10000000000L));
        } while (messageIds.contains(id));

        messageIds.add(id);

        return id;
    }

    public boolean checkRecipientCell(String recipient) {

        return recipient != null
                && (recipient.matches("^\\+27[0-9]{9}$")
                || recipient.matches("^0[0-9]{9}$"));
    }

    public boolean checkMessageLength(String message) {

        return message != null
                && !message.trim().isEmpty()
                && message.length() < 250;
    }

    public String createMessageHash(String id, int number,
                                    String message) {

        String[] words = message.trim().split("\\s+");

        return id.substring(0, 2) + ":"
                + number + ":"
                + (words[0] + words[words.length - 1]).toUpperCase();
    }

    public void sendingMessages() {

        System.out.print("How many messages would you like to send? ");

        int amount = readPositiveInt();

        for (int i = 0; i < amount; i++) {

            String id = generateMessageID();

            String recipient;

            do {
                System.out.print("Enter recipient cellphone: ");
                recipient = input.nextLine().trim();

                if (!checkRecipientCell(recipient)) {
                    System.out.println("Invalid cellphone number.");
                }

            } while (!checkRecipientCell(recipient));

            String message;

            do {
                System.out.print("Enter your message: ");
                message = input.nextLine();

                if (!checkMessageLength(message)) {
                    System.out.println("Message must be under 250 characters.");
                }

            } while (!checkMessageLength(message));

            System.out.print("Enter sender name: ");
            String sender = input.nextLine().trim();

            if (sender.isEmpty()) {
                sender = "Unknown";
            }

            int number = nextMessageNumber++;

            String hash = createMessageHash(id, number, message);

            messageIdsList.add(id);
            recipientNumber.add(recipient);
            sentMessages.add(message);
            senders.add(sender);
            msgHash.add(hash);
            messageStatus.add("Pending");

            int index = sentMessages.size() - 1;

            String status = SentMessage();

            if (status.equals("Deleted")) {
                removeMessage(index);
            } else {

                messageStatus.set(index, status);

                if (status.equals("Sent")) {
                    numMessageSent++;
                }

                if (status.equals("Stored")) {
                    saveStoredMessage(index, number);
                    loadStoredMessages();
                }
            }
        }
    }

    public String SentMessage() {

        while (true) {

            System.out.println("1. Send message");
            System.out.println("2. Disregard message");
            System.out.println("3. Store message");

            System.out.print("Choose an option: ");

            String choice = input.nextLine();

            switch (choice) {

                case "1":
                    return "Sent";

                case "2":

                    System.out.print("Enter 0 to delete or 1 to keep: ");

                    String delete = input.nextLine();

                    if (delete.equals("0")) {
                        return "Deleted";
                    }

                    disregardedMessages.add(
                            sentMessages.get(sentMessages.size() - 1));

                    return "Disregarded";

                case "3":
                    return "Stored";

                default:
                    System.out.println("Invalid choice.");
            }
        }
    }

    // Save stored messages as JSON.

    private void saveStoredMessage(int index, int number) {

        List<StoredMessage> records = readJsonMessages();

        StoredMessage record = new StoredMessage(
                messageIdsList.get(index),
                number,
                senders.get(index),
                recipientNumber.get(index),
                sentMessages.get(index),
                msgHash.get(index),
                "Stored"
        );

        records.add(record);

        writeJsonMessages(records);
    }

    private void writeJsonMessages(List<StoredMessage> records) {

        StringBuilder json = new StringBuilder("[\n");

        for (int i = 0; i < records.size(); i++) {

            StoredMessage m = records.get(i);

            json.append("  {\n")
                .append("    \"messageID\": ").append(jsonString(m.messageID)).append(",\n")
                .append("    \"messageNumber\": ").append(m.messageNumber).append(",\n")
                .append("    \"sender\": ").append(jsonString(m.sender)).append(",\n")
                .append("    \"recipient\": ").append(jsonString(m.recipient)).append(",\n")
                .append("    \"message\": ").append(jsonString(m.message)).append(",\n")
                .append("    \"messageHash\": ").append(jsonString(m.messageHash)).append(",\n")
                .append("    \"status\": ").append(jsonString(m.status)).append("\n")
                .append("  }");

            if (i < records.size() - 1) {
                json.append(",");
            }

            json.append("\n");
        }

        json.append("]\n");

        try (FileWriter writer = new FileWriter(jsonFile, StandardCharsets.UTF_8)) {

            writer.write(json.toString());

        } catch (IOException e) {
            System.out.println("Error saving JSON: " + e.getMessage());
        }
    }

    private String jsonString(String value) {

        if (value == null) {
            return "null";
        }

        StringBuilder result = new StringBuilder("\"");

        for (char c : value.toCharArray()) {

            switch (c) {

                case '"': result.append("\\\""); break;
                case '\\': result.append("\\\\"); break;
                case '\n': result.append("\\n"); break;
                case '\r': result.append("\\r"); break;
                case '\t': result.append("\\t"); break;

                default:
                    if (c < 32) {
                        result.append(String.format("\\u%04x", (int) c));
                    } else {
                        result.append(c);
                    }
            }
        }

        return result.append("\"").toString();
    }

    // Load JSON records into the stored messages array.

    public void loadStoredMessages() {

        storedMessages.clear();

        storedMessages.addAll(readJsonMessages());

        for (StoredMessage m : storedMessages) {

            messageIds.add(m.messageID);

            if (m.messageNumber >= nextMessageNumber) {
                nextMessageNumber = m.messageNumber + 1;
            }
        }
    }

    private List<StoredMessage> readJsonMessages() {

        List<StoredMessage> records = new ArrayList<>();

        if (!jsonFile.exists()) {
            return records;
        }

        try {

            String json = new String(
                    Files.readAllBytes(jsonFile.toPath()),
                    StandardCharsets.UTF_8);

            Object result = new JsonParser(json).parse();

            if (!(result instanceof List)) {
                return records;
            }

            for (Object item : (List<?>) result) {

                if (!(item instanceof Map)) {
                    continue;
                }

                Map<?, ?> m = (Map<?, ?>) item;

                records.add(new StoredMessage(
                        String.valueOf(m.get("messageID")),
                        Integer.parseInt(String.valueOf(m.get("messageNumber"))),
                        String.valueOf(m.get("sender")),
                        String.valueOf(m.get("recipient")),
                        String.valueOf(m.get("message")),
                        String.valueOf(m.get("messageHash")),
                        String.valueOf(m.get("status"))
                ));
            }

        } catch (Exception e) {
            System.out.println("Error reading JSON: " + e.getMessage());
        }

        return records;
    }

    // Stored messages menu.

    public void storedMessagesMenu() {

        int choice;

        do {

            System.out.println("\n===== STORED MESSAGES =====");
            System.out.println("1. Display senders and recipients");
            System.out.println("2. Display longest message");
            System.out.println("3. Search message by ID");
            System.out.println("4. Search messages by recipient");
            System.out.println("5. Delete message using hash");
            System.out.println("6. Display full message report");
            System.out.println("7. Return to main menu");

            System.out.print("Choose option: ");

            choice = readInt();

            switch (choice) {

                case 1:
                    displaySendersAndRecipients();
                    break;

                case 2:
                    displayLongestMessage();
                    break;

                case 3:
                    searchMessageID();
                    break;

                case 4:
                    searchMessagesByRecipient();
                    break;

                case 5:
                    deleteMessageByHash();
                    break;

                case 6:
                    displayStoredMessageReport();
                    break;

                case 7:
                    System.out.println("Returning to main menu.");
                    break;

                default:
                    System.out.println("Invalid option.");
            }

        } while (choice != 7);
    }

    public void displaySendersAndRecipients() {

        loadStoredMessages();

        for (StoredMessage m : storedMessages) {
            System.out.println("Sender: " + m.sender);
            System.out.println("Recipient: " + m.recipient);
            System.out.println("--------------------");
        }

        if (storedMessages.isEmpty()) {
            System.out.println("No stored messages.");
        }
    }

    public void displayLongestMessage() {

        loadStoredMessages();

        if (storedMessages.isEmpty()) {
            System.out.println("No stored messages.");
            return;
        }

        StoredMessage longest = storedMessages.get(0);

        for (StoredMessage m : storedMessages) {

            if (m.message.length() > longest.message.length()) {
                longest = m;
            }
        }

        System.out.println("Longest message:");
        System.out.println(longest.message);
    }

    public void searchMessageID() {

        loadStoredMessages();

        System.out.print("Enter message ID: ");
        String id = input.nextLine();

        for (StoredMessage m : storedMessages) {

            if (m.messageID.equals(id)) {

                System.out.println("Recipient: " + m.recipient);
                System.out.println("Message: " + m.message);

                return;
            }
        }

        System.out.println("Message not found.");
    }

    public void searchMessagesByRecipient() {

        loadStoredMessages();

        System.out.print("Enter recipient number: ");
        String recipient = input.nextLine();

        boolean found = false;

        for (StoredMessage m : storedMessages) {

            if (m.recipient.equals(recipient)) {

                System.out.println("Message ID: " + m.messageID);
                System.out.println("Message: " + m.message);

                found = true;
            }
        }

        if (!found) {
            System.out.println("No messages found for this recipient.");
        }
    }

    public void deleteMessageByHash() {

        loadStoredMessages();

        System.out.print("Enter message hash: ");
        String hash = input.nextLine();

        boolean removed = storedMessages.removeIf(
                m -> m.messageHash.equals(hash));

        if (removed) {

            writeJsonMessages(storedMessages);

            System.out.println("Message deleted.");

        } else {
            System.out.println("Message not found.");
        }
    }

    public void displayStoredMessageReport() {

        loadStoredMessages();

        for (StoredMessage m : storedMessages) {

            System.out.println("\nMessage ID: " + m.messageID);
            System.out.println("Message number: " + m.messageNumber);
            System.out.println("Sender: " + m.sender);
            System.out.println("Recipient: " + m.recipient);
            System.out.println("Message: " + m.message);
            System.out.println("Hash: " + m.messageHash);
            System.out.println("Status: " + m.status);

            System.out.println("--------------------");
        }

        if (storedMessages.isEmpty()) {
            System.out.println("No stored messages.");
        }
    }

    public void printMessages() {

        for (int i = 0; i < sentMessages.size(); i++) {

            System.out.println("\nMessage ID: " + messageIdsList.get(i));
            System.out.println("Sender: " + senders.get(i));
            System.out.println("Recipient: " + recipientNumber.get(i));
            System.out.println("Message: " + sentMessages.get(i));
            System.out.println("Hash: " + msgHash.get(i));
            System.out.println("Status: " + messageStatus.get(i));
        }

        if (sentMessages.isEmpty()) {
            System.out.println("No messages have been processed.");
        }
    }

    public int returnTotalMessages() {
        return numMessageSent;
    }

    private void removeMessage(int index) {

        messageIds.remove(messageIdsList.get(index));
        messageIdsList.remove(index);
        recipientNumber.remove(index);
        sentMessages.remove(index);
        senders.remove(index);
        msgHash.remove(index);
        messageStatus.remove(index);
    }

    private int readPositiveInt() {

        while (true) {

            int number = readInt();

            if (number > 0) {
                return number;
            }

            System.out.print("Enter a number greater than zero: ");
        }
    }

    private int readInt() {

        while (true) {

            try {
                return Integer.parseInt(input.nextLine().trim());

            } catch (NumberFormatException e) {
                System.out.print("Enter a valid number: ");
            }
        }
    }

    // Basic JSON parser for the JSON array and objects saved above.

    private static class JsonParser {

        private final String text;
        private int position = 0;

        JsonParser(String text) {
            this.text = text;
        }

        Object parse() {
            Object value = readValue();
            skip();
            if (position != text.length()) {
                throw new IllegalArgumentException("Extra JSON content.");
            }
            return value;
        }

        private Object readValue() {

            skip();

            if (position >= text.length()) {
                throw new IllegalArgumentException("Unexpected end of JSON.");
            }

            char c = text.charAt(position);

            if (c == '[') return readArray();
            if (c == '{') return readObject();
            if (c == '"') return readString();

            if (c == '-' || Character.isDigit(c)) {
                return readNumber();
            }

            if (text.startsWith("null", position)) {
                position += 4;
                return null;
            }

            throw new IllegalArgumentException("Invalid JSON value.");
        }

        private List<Object> readArray() {

            List<Object> list = new ArrayList<>();

            position++;
            skip();

            if (take(']')) return list;

            do {
                list.add(readValue());
                skip();
            } while (take(','));

            if (!take(']')) {
                throw new IllegalArgumentException("Expected ].");
            }

            return list;
        }

        private Map<String, Object> readObject() {

            Map<String, Object> map = new LinkedHashMap<>();

            position++;
            skip();

            if (take('}')) return map;

            while (true) {

                String key = readString();

                if (!take(':')) {
                    throw new IllegalArgumentException("Expected colon.");
                }

                map.put(key, readValue());

                if (take('}')) break;

                if (!take(',')) {
                    throw new IllegalArgumentException("Expected comma.");
                }
            }

            return map;
        }

        private String readString() {

            if (!take('"')) {
                throw new IllegalArgumentException("Expected string.");
            }

            StringBuilder value = new StringBuilder();

            while (position < text.length()) {

                char c = text.charAt(position++);

                if (c == '"') return value.toString();

                if (c == '\\') {

                    char next = text.charAt(position++);

                    switch (next) {

                        case '"': value.append('"'); break;
                        case '\\': value.append('\\'); break;
                        case 'n': value.append('\n'); break;
                        case 'r': value.append('\r'); break;
                        case 't': value.append('\t'); break;
                        case 'b': value.append('\b'); break;
                        case 'f': value.append('\f'); break;

                        case 'u':
                            value.append((char) Integer.parseInt(
                                    text.substring(position, position + 4), 16));
                            position += 4;
                            break;

                        default:
                            throw new IllegalArgumentException("Invalid escape.");
                    }

                } else {
                    value.append(c);
                }
            }

            throw new IllegalArgumentException("Unclosed JSON string.");
        }

        private Number readNumber() {

            int start = position;

            if (text.charAt(position) == '-') position++;

            while (position < text.length()
                    && Character.isDigit(text.charAt(position))) {
                position++;
            }

            return Long.parseLong(text.substring(start, position));
        }

        private boolean take(char c) {

            skip();

            if (position < text.length() && text.charAt(position) == c) {
                position++;
                return true;
            }

            return false;
        }

        private void skip() {

            while (position < text.length()
                    && Character.isWhitespace(text.charAt(position))) {
                position++;
            }
        }
    }
}
