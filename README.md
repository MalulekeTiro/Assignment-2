/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */

package com.mycompany.chat_app;

import static java.lang.Character.digit;
import java.util.HashMap;
import java.util.Scanner;

/**
 *
 * @author lab_services_student
 */
public class Chat_app {

    private String accountUsername;
    private String accountPassword;
    private String phoneNumber;
    //hashmap for storing the string values entered in the signup
    private static HashMap<String, String> accounts = new HashMap<>();

    public String getAccountUsername() {
        return accountUsername;
    }

    public void setAccountUsername(String username ) {
        accountUsername = username;
    }
    
    public String getPhoneNumber(){
        return phoneNumber;
    }
    
    public void setPhoneNumber (String number) {
        phoneNumber = number;
    }

    public String getAccountPassword() {
        return accountPassword;
    }

    public void setAccountPassword(String password) {
        accountPassword = password;
    }

     public boolean checkAccountUsername(){
         return accountUsername.contains("_") && accountUsername.length() <= 5;
     
     }  
     //cellphone validation
     public boolean checkPhoneNumber(){
         String r = "^27[0-9]{10}$";
        if (phoneNumber.matches(r)) {
            System.out.println("Valid Mobile Number");
            return true;
        }
        else {
            System.out.println("Invalid Mobile Number");
            return false;
        }
      
     }
    
    public boolean checkAccountPassword(){
        boolean upperC = false;
        boolean lowerC = false;
        boolean digit = false;
        boolean special = false;
        
        if (accountPassword.length() < 8){
           return false;
        }
        //password validation
        for (int lcv = 0; lcv < accountPassword.length(); lcv++){
             char ch = accountPassword.charAt(lcv);
             
             if (Character.isUpperCase(ch)){
                 upperC = true;
             }else if (Character.isLowerCase(ch)){
                 lowerC = true;
             }else if (Character.isDigit(ch)){
                 digit = true;
             } else{
                 special = true;
             }
        }
 
        return upperC && lowerC && digit && special;
    }
            
    public static void main(String[] args) {
        
        Chat_app account1 = new Chat_app();
        Scanner scanner = new Scanner(System.in);
        //login while loop to loop until correct credentials are entered
        loginLoop:
        while (true) {
            System.out.print("\nPress 1 to login or press 2 to sign up: ");
            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {

                // ── LOGIN ──────────────────────────────────────────────────
                case 1:
                    System.out.println("\n=== Login ===");
                    System.out.print("Enter username: ");
                    String uname = scanner.nextLine();
                    account1.setAccountUsername(uname);

                    System.out.print("Enter password: ");
                    String pword = scanner.nextLine();
                    account1.setAccountPassword(pword);

                    if (accounts.containsKey(uname) && accounts.get(uname).equals(pword)) {
                        System.out.println("\nWelcome, " + uname + ". It is great to see you again.");

                        // Load stored messages from file on every login
                        messages.loadStoredMessages("stored_messages.json");

                        System.out.println("Welcome to QuickChat");
                        System.out.print("How many messages would you like to send? ");
                        int numMessages = scanner.nextInt();
                        scanner.nextLine();
                        int messagesSentThisSession = 0;

                        messageLoop:
                        while (true) {
                            System.out.println("\n--- QuickChat Menu ---");
                            System.out.println("1) Send Messages");
                            System.out.println("2) Show recently sent messages");
                            System.out.println("3) Stored Messages");
                            System.out.println("4) Quit");
                            System.out.print("Choice: ");
                            int menuChoice = scanner.nextInt();
                            scanner.nextLine();

                            switch (menuChoice) {

                                // ── 1) Send ───────────────────────────────
                                case 1:
                                    if (messagesSentThisSession >= numMessages) {
                                        System.out.println("You have reached your message limit of "
                                                           + numMessages + ".");
                                        break;
                                    }
                                    System.out.print("Enter recipient cell number (e.g. 27XXXXXXXXXX): ");
                                    String recipient = scanner.nextLine();

                                    messages tempMsg = new messages(recipient, "", messagesSentThisSession + 1);
                                    String cellCheck = tempMsg.checkReceiverCell();
                                    if (!cellCheck.equals("Cell phone number successfully captured.")) {
                                        System.out.println(cellCheck);
                                        break;
                                    }

                                    System.out.print("Enter your message (max 250 characters): ");
                                    String msgText = scanner.nextLine();
                                    if (msgText.length() > 250) {
                                        System.out.println("Please enter a message of less than 250 characters.");
                                        break;
                                    }

                                    messages newMsg = new messages(recipient, msgText,
                                                                   messagesSentThisSession + 1);
                                    String result = newMsg.sentMessage(scanner);
                                    System.out.println(result);

                                    if (result.equals("Message successfully sent")
                                     || result.equals("Message successfully stored")) {
                                        messagesSentThisSession++;
                                    }

                                    if (messagesSentThisSession >= numMessages) {
                                        System.out.println("\nAll messages sent.");
                                        System.out.println("Total messages sent: "
                                                           + messages.returnTotalMessages());
                                        break messageLoop;
                                    }
                                    break;

                                // ── 2) Recently sent ──────────────────────
                                case 2:
                                    System.out.println(messages.printMessages());
                                    break;

                                // ── 3) Stored Messages sub-menu ───────────
                                case 3:
                                    storedMessagesMenu(scanner);
                                    break;

                                // ── 4) Quit ───────────────────────────────
                                case 4:
                                    System.out.println("Total messages sent: "
                                                       + messages.returnTotalMessages());
                                    break messageLoop;

                                default:
                                    System.out.println("Invalid option. Please try again.");
                            }
                        }
                        break loginLoop;

                    } else if (!accounts.containsKey(uname)) {
                        System.out.println("Username is incorrect, does not exist.");
                    } else {
                        System.out.println("Incorrect password, please try again.");
                    }
                    break;

                // ── SIGN UP ────────────────────────────────────────────────
                case 2:
                    System.out.println("\n=== Sign Up ===");

                    System.out.print("Enter phone number (27XXXXXXXXXX): ");
                    String phone = scanner.nextLine();
                    account1.setPhoneNumber(phone);
                    if (account1.checkPhoneNumber()) {
                        System.out.println("Cell phone number successfully added.");
                    } else {
                        System.out.println("Cell phone number is incorrectly formatted or does not"
                                           + " include an international country code.");
                        return;
                    }

                    System.out.print("Enter username (must contain _ and be ≤5 chars): ");
                    String newUser = scanner.nextLine();
                    account1.setAccountUsername(newUser);
                    if (account1.checkAccountUsername()) {
                        System.out.println("Username successfully captured.");
                    } else {
                        System.out.println("Username is incorrectly formatted. Please ensure your "
                                           + "username contains an underscore and is no more than "
                                           + "five characters long.");
                        return;
                    }

                    System.out.print("Enter password (≥8 chars, upper, digit, special): ");
                    String newPass = scanner.nextLine();
                    account1.setAccountPassword(newPass);
                    if (account1.checkAccountPassword()) {
                        System.out.println("Password successfully captured.");
                    } else {
                        System.out.println("Password is incorrectly formatted. Please ensure the "
                                           + "password contains at least eight characters, a capital"
                                           + " letter, a number, and a special character.");
                        return;
                    }

                    accounts.put(newUser, newPass);
                    System.out.println("Account successfully created for " + newUser + ".");
                    break;

                default:
                    System.out.println("Error: Please enter 1 or 2.");
            }
        }
    }
    
    // ── Stored Messages sub-menu ───────────────────────────────────────────────

    private static void storedMessagesMenu(Scanner scanner) {
        boolean running = true;
        while (running) {
            System.out.println("\n--- Stored Messages Menu ---");
            System.out.println("a) Display sender and recipient of all stored messages");
            System.out.println("b) Display the longest message");
            System.out.println("c) Search for a message by ID");
            System.out.println("d) Search messages by recipient");
            System.out.println("e) Delete a message using its hash");
            System.out.println("f) Display full messages report");
            System.out.println("0) Back to main menu");
            System.out.print("Choice: ");
            String sub = scanner.nextLine().trim().toLowerCase();

            switch (sub) {
                case "a":
                    System.out.println("\n" + messages.displayStoredMessagesSummary());
                    break;

                case "b":
                    System.out.println("\nLongest message:\n" + messages.findLongestMessage());
                    break;

                case "c":
                    System.out.print("Enter message ID to search: ");
                    String id = scanner.nextLine().trim();
                    System.out.println("\nResult: " + messages.searchMessageByID(id));
                    break;

                case "d":
                    System.out.print("Enter recipient number to search: ");
                    String rec = scanner.nextLine().trim();
                    System.out.println("\nMessages for " + rec + ":\n"
                                       + messages.searchMessagesByRecipient(rec));
                    break;

                case "e":
                    System.out.print("Enter message hash to delete: ");
                    String hash = scanner.nextLine().trim();
                    System.out.println("\n" + messages.deleteMessageByHash(hash));
                    break;

                case "f":
                    System.out.println("\n" + messages.displayReport());
                    break;

                case "0":
                    running = false;
                    break;

                default:
                    System.out.println("Invalid option. Please enter a–f or 0.");
            }
        }
    }
}
