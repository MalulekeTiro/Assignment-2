<img width="1919" height="936" alt="image" src="https://github.com/user-attachments/assets/ac41a9cf-f4a5-43ee-937a-3fa3faf26af9" />



public String checkMessageLength() {
    if (messageText.length() <= 250) {
        return "Message ready to send.";
    }
    int excess = messageText.length() - 250;
    return "Message exceeds 250 characters by " + excess + "; please reduce the size.";
}

public String createMessageHash() {
    String firstTwo = messageID.substring(0, 2);
    String[] words = messageText.trim().split("\\s+");
    String firstWord = words[0].replaceAll("[^a-zA-Z]", "").toUpperCase();
    String lastWord = words[words.length - 1].replaceAll("[^a-zA-Z]", "").toUpperCase();
    return firstTwo + ":" + messageNumber + ":" + firstWord + (words.length > 1 ? lastWord : "");
}

public String checkReceiverCell() {
    if (receiver != null && receiver.matches("^27[0-9]{10}$")) {
        return "Cell phone number successfully captured.";
    }
    return "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.";
}
