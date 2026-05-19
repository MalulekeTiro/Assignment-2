@Test
public void messageLength_valid() {
    messages msg = new messages("271234567890", "Hi Mike, can you join us for dinner tonight?", 0);
    String expectedValue = "Message ready to send.";
    String actualValue = msg.checkMessageLength();
    Assertions.assertEquals(expectedValue, actualValue);
    System.out.println("Message is within the 250 character limit.");
}

// Recipient cell tests - all use 271234567890 for valid
@Test
public void recipientCell_valid() {
    messages msg = new messages("271234567890", "Hi Mike, can you join us for dinner tonight?", 0);
    String expectedValue = "Cell phone number successfully captured.";
    String actualValue = msg.checkReceiverCell();
    Assertions.assertEquals(expectedValue, actualValue);
    System.out.println("Recipient cell number is correctly formatted.");
}

@Test
public void recipientCell_invalid() {
    messages msg = new messages("08575975889", "Hi Keegan, did you receive the payment?", 0);
    String expectedValue = "Cell phone number is incorrectly formatted or does not contain an international code. Please correct the number and try again.";
    String actualValue = msg.checkReceiverCell();
    Assertions.assertEquals(expectedValue, actualValue);
    System.out.println("Recipient cell number is incorrectly formatted.");
}

@Test
public void recipientCell_True() {
    messages msg = new messages("271234567890", "Hi Mike, can you join us for dinner tonight?", 0);
    Assertions.assertTrue(msg.checkReceiverCell().equals("Cell phone number successfully captured."));
}

@Test
public void recipientCell_False() {
    messages msg = new messages("08575975889", "Hi Keegan, did you receive the payment?", 0);
    Assertions.assertFalse(msg.checkReceiverCell().equals("Cell phone number successfully captured."));
}

// Hash test - fix recipient number here too
@Test
public void messageHash_correct() {
    messages msg = new messages("271234567890", "Hi Mike, can you join us for dinner tonight?", 0);
    String hash = msg.createMessageHash();
    Assertions.assertTrue(hash.endsWith(":0:HITONIGHT"));
    System.out.println("Message hash: " + hash);
}

@Test
public void messageHash_loop() {
    String[] recipients   = {"271234567890", "271234567890"};
    String[] messages_    = {
        "Hi Mike, can you join us for dinner tonight?",
        "Hi Keegan, did you receive the payment?"
    };
    String[] expectedEnds = {":0:HITONIGHT", ":1:HIPAYMENT"};

    for (int i = 0; i < recipients.length; i++) {
        messages msg = new messages(recipients[i], messages_[i], i);
        String hash = msg.createMessageHash();
        Assertions.assertTrue(hash.endsWith(expectedEnds[i]),
            "Hash failed for message " + (i + 1) + ": " + hash);
        System.out.println("Message " + (i + 1) + " hash: " + hash);
    }
}
