
package com.mycompany.chat_app;

import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Random;
import java.util.Scanner;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 *
 * @author lab_services_student
 */


public class messages {
    private String messageID;
    private String receiver;
    private String messageText;
    private String messageHash;
    private int messageNumber;
    
    private static ArrayList<messages> sentMessages = new ArrayList<>();
    private static int totalMessagesSent = 0;
    
    private static ArrayList<messages> disregardedMessages = new ArrayList<>();
    private static ArrayList<messages> storedMessages      = new ArrayList<>();
    private static ArrayList<String>   messageHashList     = new ArrayList<>();
    private static ArrayList<String>   messageIDList       = new ArrayList<>();
    
    public messages(String receiver, String messageText, int messageNumber){
        this.receiver = receiver;
        this.messageText = messageText;
        this.messageNumber = messageNumber;
        this.messageID = generateMessageID();
        this.messageHash = createMessageHash();
    }
    
    public messages(String receiver, String messageText, int messageNumber, String customMessageID) {
        this.receiver      = receiver;
        this.messageText   = messageText;
        this.messageNumber = messageNumber;
        this.messageID     = (customMessageID != null && !customMessageID.isEmpty())
                             ? customMessageID : generateMessageID();
        this.messageHash   = createMessageHash();
    }
    
    private String generateMessageID(){
        Random rand = new Random();
        long id = (long) (rand.nextDouble() * 9_000_000_000L) + 1_000_000_000L;
        return String.valueOf(id);
    }
    
    public boolean checkMessageID(){
        return messageID != null && messageID.length() <= 10;
    }
    
    public String checkReceiverCell(){
    if (receiver != null && receiver.matches("^27[0-9]{10}$")){
        return "Cell number validated.";
        }
        return "Cell number is incorrect. Please enter the correct number";
    }
    
    public String createMessageHash(){
        String firstTwo = messageID.substring(0, 2);
        String[] words = messageText.trim().split("\\s+");
        String firstWord = words[0].toUpperCase();
        String lastWord = words[words.length - 1].toUpperCase();
        
        return firstTwo + ":" + messageNumber + ":" + firstWord + (words.length > 1 ? lastWord : "");
    }
    
    public String sentMessage(Scanner scanner){
        System.out.println("\nChoose an option:");
        System.out.println("1) Send Message");
        System.out.println("2) Disregard Message");
        System.out.println("3) Store Message to send later");
        int choice = scanner.nextInt();
        scanner.nextLine();
        
        switch (choice) {
            case 1:
                sentMessages.add(this);
                messageIDList.add(this.messageID);
                messageHashList.add(this.messageHash);
                totalMessagesSent++;
                
                System.out.println("\n--- Message Details ---");
                System.out.println("Message ID: " + messageID);
                System.out.println("Message Hash: " + messageHash);
                System.out.println("Message Recipient: " + receiver);
                System.out.println("Message: " + messageText);
                System.out.println("----------------------");
                return "Message successfully sent";
            case 2:
                disregardedMessages.add(this);
                messageIDList.add(this.messageID);
                messageHashList.add(this.messageHash);
                return "Press 0 to delete the message";
            case 3:
                storeMessage();
                storedMessages.add(this);
                messageIDList.add(this.messageID);
                messageHashList.add(this.messageHash);
                return "Message successfully stored";
            default: 
                return "Invalid option. Message was not sent";
        }
    }
    
    public static String printMessages(){
        if (sentMessages.isEmpty()){
            return "No messages have been sent.";
        }
        StringBuilder sb = new StringBuilder();
        for (messages m : sentMessages) {
            sb.append("Message ID: ").append(m.messageID).append("\n");
            sb.append("Message hash: ").append(m.messageHash).append("\n");
            sb.append("Recipient: ").append(m.receiver).append("\n");
            sb.append("Message: ").append(m.messageText).append("\n\n");
        }
        return sb.toString();
    }
    
    public static int returnTotalMessages(){
        return totalMessagesSent;
    }
    
    public void storeMessage(){
        try (FileWriter fw = new FileWriter("stored_messages.json", true)){
            String json = "{\n"
                    + " \"messageID\": \"" + messageID + "\",\n" 
                    + " \"messageHash\": \"" + messageHash + "\",\n"
                    + " \"Recipient\": \"" + receiver + "\",\n"
                    + " \"message\": \"" + messageText.replace("\"", "\\\"") + "\",\n"
                    + " \"messageNumber\": \"" + messageNumber + "\"\n"
                    + "},\n";
            fw.write(json);
            System.out.println("message stored to stored_messages.json");
        } catch (IOException e) {
            System.out.println("error storing message: " + e.getMessage());
        }
    }
}

public static String displayStoredMessagesSummary() {
    if (storedMessages.isEmpty()) {
        return "No stored messages available.";
    }
    StringBuilder sb = new StringBuilder("=== Stored Messages ===\n");
    for (messages m : storedMessages) {
        sb.append("Recipient: ").append(m.receiver)
          .append("\nMessage:   ").append(m.messageText).append("\n\n");
    }
    return sb.toString().trim();
}


public static void loadStoredMessages(String filename) {
    try {
        String content = new String(Files.readAllBytes(Paths.get(filename)));
        // [^{}]* matches any char except braces, including newlines (char-class negation)
        Pattern pattern = Pattern.compile("\\{[^{}]*\\}");
        Matcher matcher = pattern.matcher(content);
        while (matcher.find()) {
            String block     = matcher.group();
            String msgID     = extractJsonValue(block, "messageID");
            String msgHash   = extractJsonValue(block, "messageHash");
            String recipient = extractJsonValue(block, "Recipient");
            String msgText   = extractJsonValue(block, "message");
            String numStr    = extractJsonValue(block, "messageNumber");
            int msgNum = 0;
            try { if (numStr != null) msgNum = Integer.parseInt(numStr); }
            catch (NumberFormatException ignored) {}

            if (recipient != null && msgText != null) {
                messages m = new messages(recipient, msgText, msgNum, msgID);
                if (msgHash != null) m.messageHash = msgHash;
                storedMessages.add(m);
                if (msgID   != null) messageIDList.add(msgID);
                if (msgHash != null) messageHashList.add(msgHash);
            }
        }
    } catch (IOException e) {
        System.out.println("Could not load stored messages: " + e.getMessage());
    }
}

private static String extractJsonValue(String json, String key) {
    Pattern p = Pattern.compile(
        "\"" + Pattern.quote(key) + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\"");
    Matcher m = p.matcher(json);
    return m.find() ? m.group(1).replace("\\\"", "\"") : null;
}

 public static void addToSentMessages(messages msg) {
        sentMessages.add(msg);
        messageIDList.add(msg.messageID);
        messageHashList.add(msg.messageHash);
        totalMessagesSent++;
    }

    public static void addToDisregardedMessages(messages msg) {
        disregardedMessages.add(msg);
        messageIDList.add(msg.messageID);
        messageHashList.add(msg.messageHash);
    }

    public static void addToStoredMessages(messages msg) {
        storedMessages.add(msg);
        messageIDList.add(msg.messageID);
        messageHashList.add(msg.messageHash);
    }
    
    public static void clearAllMessages() {
        sentMessages.clear();
        disregardedMessages.clear();
        storedMessages.clear();
        messageIDList.clear();
        messageHashList.clear();
        totalMessagesSent = 0;
    }
    
    public String getMessageID()   { return messageID; }
    public String getMessageHash() { return messageHash; }
    public String getReceiver()    { return receiver; }
    public String getMessageText() { return messageText; }
    public int    getMessageNumber(){ return messageNumber; }

    public static ArrayList<messages> getSentMessages()        { return sentMessages; }
    public static ArrayList<messages> getDisregardedMessages() { return disregardedMessages; }
    public static ArrayList<messages> getStoredMessages()      { return storedMessages; }
    public static ArrayList<String>   getMessageHashList()     { return messageHashList; }
    public static ArrayList<String>   getMessageIDList()       { return messageIDList; }
    public static int                 returnTotalMessages()     { return totalMessagesSent; }
    
    public static ArrayList<String> getSentMessageTexts() {
        ArrayList<String> texts = new ArrayList<>();
        for (messages m : sentMessages) {
            texts.add(m.messageText);
        }
        return texts;
    }
    
    public static String printMessages() {
        if (sentMessages.isEmpty()) {
            return "No messages have been sent.";
        }
        StringBuilder sb = new StringBuilder();
        for (messages m : sentMessages) {
            sb.append("Message ID: ")   .append(m.messageID)  .append("\n");
            sb.append("Message hash: ") .append(m.messageHash).append("\n");
            sb.append("Recipient: ")    .append(m.receiver)   .append("\n");
            sb.append("Message: ")      .append(m.messageText).append("\n\n");
        }
        return sb.toString();
    }
    
    public static String findLongestMessage() {
        ArrayList<messages> all = new ArrayList<>();
        all.addAll(sentMessages);
        all.addAll(storedMessages);
        all.addAll(disregardedMessages);

        if (all.isEmpty()) {
            return "No messages available.";
        }
        messages longest = all.get(0);
        for (messages m : all) {
            if (m.messageText.length() > longest.messageText.length()) {
                longest = m;
            }
        }
        return longest.messageText;
    }
    
    public static String searchMessageByID(String id) {
        ArrayList<messages> all = new ArrayList<>();
        all.addAll(sentMessages);
        all.addAll(storedMessages);
        all.addAll(disregardedMessages);

        for (messages m : all) {
            if (m.messageID.equals(id)) {
                return m.messageText;
            }
        }
        return "No message found with ID: " + id;
    }
    
    public static ArrayList<String> getMessagesByRecipient(String recipient) {
        ArrayList<String> results = new ArrayList<>();
        ArrayList<messages> all   = new ArrayList<>();
        all.addAll(sentMessages);
        all.addAll(storedMessages);
        all.addAll(disregardedMessages);

        for (messages m : all) {
            if (m.receiver.equals(recipient)) {
                results.add(m.messageText);
            }
        }
        return results;
    }
    
    public static String searchMessagesByRecipient(String recipient) {
        ArrayList<String> results = getMessagesByRecipient(recipient);
        if (results.isEmpty()) {
            return "No messages found for recipient: " + recipient;
        }
        StringBuilder sb = new StringBuilder();
        for (String text : results) {
            sb.append(text).append("\n");
        }
        return sb.toString().trim();
    }
    
    public static String deleteMessageByHash(String hash) {
        for (int i = 0; i < sentMessages.size(); i++) {
            if (sentMessages.get(i).messageHash.equals(hash)) {
                String text = sentMessages.get(i).messageText;
                sentMessages.remove(i);
                messageHashList.remove(hash);
                return "Message: \"" + text + "\" successfully deleted.";
            }
        }
        for (int i = 0; i < storedMessages.size(); i++) {
            if (storedMessages.get(i).messageHash.equals(hash)) {
                String text = storedMessages.get(i).messageText;
                storedMessages.remove(i);
                messageHashList.remove(hash);
                return "Message: \"" + text + "\" successfully deleted.";
            }
        }
        for (int i = 0; i < disregardedMessages.size(); i++) {
            if (disregardedMessages.get(i).messageHash.equals(hash)) {
                String text = disregardedMessages.get(i).messageText;
                disregardedMessages.remove(i);
                messageHashList.remove(hash);
                return "Message: \"" + text + "\" successfully deleted.";
            }
        }
        return "No message found with hash: " + hash;
    }
    
    public static String displayReport() {
        if (sentMessages.isEmpty()) {
            return "No sent messages to report.";
        }
        StringBuilder sb = new StringBuilder("=== Sent Messages Report ===\n");
        for (messages m : sentMessages) {
            sb.append("Message Hash: ").append(m.messageHash).append("\n");
            sb.append("Recipient: ")  .append(m.receiver)   .append("\n");
            sb.append("Message: ")    .append(m.messageText).append("\n");
            sb.append("--------------------------\n");
        }
        return sb.toString();
    }

