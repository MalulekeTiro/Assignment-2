@Test
    public void messageLength_valid() {
        messages msg = new messages("27718693002", "Hi Mike, can you join us for dinner tonight?", 0);
        String expectedValue = "Message ready to send.";
        String actualValue = msg.checkMessageLength();
        Assertions.assertEquals(expectedValue, actualValue);
        System.out.println("Message is within the 250 character limit.");
    }

    @Test
    public void messageLength_invalid() {
        // 260 character message — exceeds by 10
        String longMessage = "a".repeat(260);
        messages msg = new messages("27718693002", longMessage, 0);
        String expectedValue = "Message exceeds 250 characters by 10; please reduce the size.";
        String actualValue = msg.checkMessageLength();
        Assertions.assertEquals(expectedValue, actualValue);
        System.out.println("Message exceeds the 250 character limit.");
    }

    @Test
    public void messageLength_True() {
        messages msg = new messages("27718693002", "Hi Mike, can you join us for dinner tonight?", 0);
        Assertions.assertTrue(msg.checkMessageLength().equals("Message ready to send."));
    }

    @Test
    public void messageLength_False() {
        String longMessage = "a".repeat(260);
        messages msg = new messages("27718693002", longMessage, 0);
        Assertions.assertFalse(msg.checkMessageLength().equals("Message ready to send."));
    }

    // ── Recipient Cell Number Tests ───────────────────────────────────

    @Test
    public void recipientCell_valid() {
        messages msg = new messages("27718693002", "Hi Mike, can you join us for dinner tonight?", 0);
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
        messages msg = new messages("27718693002", "Hi Mike, can you join us for dinner tonight?", 0);
        Assertions.assertTrue(msg.checkReceiverCell().equals("Cell phone number successfully captured."));
    }

    @Test
    public void recipientCell_False() {
        messages msg = new messages("08575975889", "Hi Keegan, did you receive the payment?", 0);
        Assertions.assertFalse(msg.checkReceiverCell().equals("Cell phone number successfully captured."));
    }

    // ── Message Hash Tests ────────────────────────────────────────────

    @Test
    public void messageHash_correct() {
        // Message 1 test data: "Hi Mike, can you join us for dinner tonight?" with messageNumber 0
        // Expected hash ends with :0:HITONIGHT (first two digits are random)
        messages msg = new messages("27718693002", "Hi Mike, can you join us for dinner tonight?", 0);
        String hash = msg.createMessageHash();
        Assertions.assertTrue(hash.endsWith(":0:HITONIGHT"));
        System.out.println("Message hash: " + hash);
    }

    @Test
    public void messageHash_loop() {
        // Tests all message hashes match the correct format: XX:N:FIRSTLAST
        String[] recipients  = {"27718693002", "27718693002"};
        String[] messages_   = {
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
}
