package com.mycompany.chat_app;

import java.util.HashMap;
import java.util.Scanner;

public class Chat_app {

    private String accountUsername;
    private String accountPassword;
    private String phoneNumber;
    private static HashMap<String, String> accounts = new HashMap<>();

    public String getAccountUsername() {
        return accountUsername;
    }

    public void setAccountUsername(String username) {
        accountUsername = username;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public void setPhoneNumber(String number) {
        phoneNumber = number;
    }

    public String getAccountPassword() {
        return accountPassword;
    }

    public void setAccountPassword(String password) {
        accountPassword = password;
    }

    public boolean checkAccountUsername() {
        return accountUsername.contains("_") && accountUsername.length() <= 5;
    }

    public boolean checkPhoneNumber() {
        String r = "^27[0-9]{10}$";
        if (phoneNumber.matches(r)) {
            System.out.println("Valid Mobile Number");
            return true;
        } else {
            System.out.println("Invalid Mobile Number");
            return false;
        }
    }

    public boolean checkAccountPassword() {
        boolean upperC = false;
        boolean lowerC = false;
        boolean digit = false;
        boolean special = false;

        if (accountPassword.length() < 8) {
            return false;
        }

        for (int lcv = 0; lcv < accountPassword.length(); lcv++) {
            char ch = accountPassword.charAt(lcv);

            if (Character.isUpperCase(ch)) {
                upperC = true;
            } else if (Character.isLowerCase(ch)) {
                lowerC = true;
            } else if (Character.isDigit(ch)) {
                digit = true;
            } else {
                special = true;
            }
        }

        return upperC && lowerC && digit && special;
    }

    public static void main(String[] args) {

        Chat_app account1 = new Chat_app();
        Scanner scanner = new Scanner(System.in);

        loginLoop:
        while (true) {
            System.out.print("Press 1 to login or press 2 to sign up: ");
            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1:
                    System.out.println("Login");
                    System.out.print("Enter username: ");
                    String accountUsername = scanner.nextLine();
                    account1.setAccountUsername(accountUsername);

                    System.out.print("Enter password: ");
                    String accountPassword = scanner.nextLine();
                    account1.setAccountPassword(accountPassword);

                    if (accounts.containsKey(accountUsername) && accounts.get(accountUsername).equals(accountPassword)) {
                        System.out.println("Welcome, " + accountUsername + ". It is great to see you again.");

                        // ── POST-LOGIN MESSAGING SECTION ──────────────────────────────
                        System.out.println("Welcome to QuickChat.");
                        System.out.print("How many messages would you like to send? ");
                        int numMessages = scanner.nextInt();
                        scanner.nextLine();
                        int messagesSentThisSession = 0;

                        appLoop:
                        while (true) {
                            System.out.println("\n--- QuickChat Menu ---");
                            System.out.println("1) Send Messages");
                            System.out.println("2) Show recently sent messages");
                            System.out.println("3) Quit");
                            System.out.print("Choose an option: ");
                            int menuChoice = scanner.nextInt();
                            scanner.nextLine();

                            switch (menuChoice) {
                                case 1:
                                    if (messagesSentThisSession >= numMessages) {
                                        System.out.println("You have reached your message limit of " + numMessages + ".");
                                        break;
                                    }

                                    // Collect and validate recipient
                                    System.out.print("Enter recipient cell number (10 digits with country code): ");
                                    String recipient = scanner.nextLine();

                                    Message tempMsg = new Message(recipient, "", messagesSentThisSession + 1);
                                    String cellCheck = tempMsg.checkRecipientCell();
                                    if (!cellCheck.equals("Cell number validated.")) {
                                        System.out.println(cellCheck);
                                        break;
                                    }

                                    // Collect and validate message text
                                    System.out.print("Enter your message (max 250 characters): ");
                                    String messageText = scanner.nextLine();
                                    if (messageText.length() > 250) {
                                        System.out.println("Please enter a message of less than 250 characters.");
                                        break;
                                    }

                                    // Build the full validated message object
                                    Message newMessage = new Message(recipient, messageText, messagesSentThisSession + 1);
                                    System.out.println("Message sent");

                                    // Prompt user: send / disregard / store
                                    String result = newMessage.sentMessage();
                                    System.out.println(result);

                                    if (result.equals("Message successfully sent") || result.equals("Message successfully stored")) {
                                        messagesSentThisSession++;
                                    }

                                    // Auto-exit once message limit is reached
                                    if (messagesSentThisSession >= numMessages) {
                                        System.out.println("\nAll messages sent.");
                                        System.out.println("Total messages sent: " + Message.returnTotalMessages());
                                        break appLoop;
                                    }
                                    break;

                                case 2:
                                    System.out.println("Coming Soon.");
                                    break;

                                case 3:
                                    System.out.println("Total messages sent: " + Message.returnTotalMessages());
                                    break appLoop;

                                default:
                                    System.out.println("Invalid option. Please try again.");
                            }
                        }
                        // ── END OF MESSAGING SECTION ──────────────────────────────────

                        break loginLoop;

                    } else if (!accounts.containsKey(accountUsername)) {
                        System.out.println("Username is incorrect.");
                    } else {
                        System.out.println("Incorrect password.");
                    }
                    break;

                case 2:
                    System.out.println("Signup");

                    System.out.print("Enter phone number: ");
                    String phoneNumber = scanner.nextLine();
                    account1.setPhoneNumber(phoneNumber);
                    if (account1.checkPhoneNumber()) {
                        System.out.println("Cell phone number successfully added.");
                    } else {
                        System.out.println("Cell phone number is incorrectly formatted or does not include an international country code.");
                        return;
                    }

                    System.out.print("Enter username: ");
                    accountUsername = scanner.nextLine();
                    account1.setAccountUsername(accountUsername);
                    if (account1.checkAccountUsername()) {
                        System.out.println("Username successfully captured.");
                    } else {
                        System.out.println("Username is incorrectly formatted. Please ensure your username contains an underscore and is no more than five characters long.");
                        return;
                    }

                    System.out.print("Enter password: ");
                    accountPassword = scanner.nextLine();
                    account1.setAccountPassword(accountPassword);
                    if (account1.checkAccountPassword()) {
                        System.out.println("Password successfully captured.");
                    } else {
                        System.out.println("Password is incorrectly formatted. Please ensure the password contains at least eight characters, a capital letter, a number, and a special character.");
                        return;
                    }

                    accounts.put(accountUsername, accountPassword);
                    System.out.println("Account successfully created for " + accountUsername);
                    break;

                default:
                    System.out.println("Error: Please try again!");
            }
        }
    }
}




