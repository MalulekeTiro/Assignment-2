@Test
public void messageLength_invalid() {
    String longMessage = "a".repeat(260);
    messages msg = new messages("271234567890", longMessage, 0);
    String expectedValue = "Message exceeds 250 characters by 10; please reduce the size.";
    String actualValue = msg.checkMessageLength();
    Assertions.assertEquals(expectedValue, actualValue);
    System.out.println("Message exceeds the 250 character limit.");
}

@Test
public void messageHash_incorrect() {
    // Wrong message number and wrong message — hash should NOT end with :0:HITONIGHT
    messages msg = new messages("271234567890", "Hello World", 5);
    String hash = msg.createMessageHash();
    Assertions.assertFalse(hash.endsWith(":0:HITONIGHT"),
        "Hash should not match :0:HITONIGHT but was: " + hash);
    System.out.println("Message hash is incorrect as expected: " + hash);
}

@Test
public void messageHash_loop_incorrect() {
    // Wrong expected endings — tests should all fail the assertFalse
    String[] recipients   = {"271234567890", "271234567890"};
    String[] messages_    = {
        "Hi Mike, can you join us for dinner tonight?",
        "Hi Keegan, did you receive the payment?"
    };
    String[] wrongEnds = {":0:BYETONIGHT", ":1:HIPAYMENT123"};

    for (int i = 0; i < recipients.length; i++) {
        messages msg = new messages(recipients[i], messages_[i], i);
        String hash = msg.createMessageHash();
        Assertions.assertFalse(hash.endsWith(wrongEnds[i]),
            "Hash should not match " + wrongEnds[i] + " but was: " + hash);
        System.out.println("Message " + (i + 1) + " hash incorrectly formatted as expected: " + hash);
    }
}
