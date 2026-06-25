
import com.mycompany.chat_app.messages;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import java.util.ArrayList;

public class messagetests {

    // =========================================================================
    // SETUP — runs before EVERY test to give each one a clean, known state
    // =========================================================================
    @BeforeEach
    public void setupTestData() {
        // Clear all static arrays first so nothing leaks between tests
        messages.clearAllMessages();

        // Message 1 – Flag: Sent
        messages msg1 = new messages("+27834557896", "Did you get the cake?", 1);
        messages.addToSentMessages(msg1);

        // Message 2 – Flag: Stored
        messages msg2 = new messages("+27838884567",
            "Where are you? You are late! I have asked you to be on time.", 2);
        messages.addToStoredMessages(msg2);

        // Message 3 – Flag: Disregard
        messages msg3 = new messages("+27834484567", "Yohoooo, I am at your gate.", 3);
        messages.addToDisregardedMessages(msg3);

        // Message 4 – Flag: Sent  (developer supplies fixed ID so searchByID is predictable)
        messages msg4 = new messages("0838884567", "It is dinner time !", 4, "0838884567");
        messages.addToSentMessages(msg4);

        // Message 5 – Flag: Stored
        messages msg5 = new messages("+27838884567", "Ok, I am leaving without you.", 5);
        messages.addToStoredMessages(msg5);
    }

    // =========================================================================
    // PART 2 TESTS
    // =========================================================================

    @Test
    public void messageLength_valid() {
        messages msg = new messages("271234567890",
            "Hi Mike, can you join us for dinner tonight?", 0);
        String expected = "Message ready to send.";
        String actual   = msg.checkMessageLength();
        Assertions.assertEquals(expected, actual);
        System.out.println("Message is within the 250 character limit.");
    }

    @Test
    public void messageLength_invalid() {
        String longMsg = "a".repeat(260);
        messages msg = new messages("271234567890", longMsg, 0);
        String expected = "Message exceeds 250 characters by 10; please reduce the size.";
        String actual   = msg.checkMessageLength();
        Assertions.assertEquals(expected, actual);
        System.out.println("Message exceeds the 250 character limit.");
    }

    @Test
    public void recipientCell_valid() {
        messages msg = new messages("271234567890",
            "Hi Mike, can you join us for dinner tonight?", 0);
        String expected = "Cell phone number successfully captured.";
        String actual   = msg.checkReceiverCell();
        Assertions.assertEquals(expected, actual);
        System.out.println("Recipient cell number is correctly formatted.");
    }

    @Test
    public void recipientCell_invalid() {
        messages msg = new messages("08575975889",
            "Hi Keegan, did you receive the payment?", 0);
        String expected = "Cell phone number is incorrectly formatted or does not contain an "
                        + "international code. Please correct the number and try again.";
        String actual = msg.checkReceiverCell();
        Assertions.assertEquals(expected, actual);
        System.out.println("Recipient cell number is incorrectly formatted.");
    }

    @Test
    public void recipientCell_True() {
        messages msg = new messages("271234567890",
            "Hi Mike, can you join us for dinner tonight?", 0);
        Assertions.assertTrue(
            msg.checkReceiverCell().equals("Cell phone number successfully captured."));
    }

    @Test
    public void recipientCell_False() {
        messages msg = new messages("08575975889",
            "Hi Keegan, did you receive the payment?", 0);
        Assertions.assertFalse(
            msg.checkReceiverCell().equals("Cell phone number successfully captured."));
    }

    @Test
    public void messageHash_correct() {
        messages msg = new messages("271234567890",
            "Hi Mike, can you join us for dinner tonight?", 0);
        String hash = msg.createMessageHash();
        Assertions.assertTrue(hash.endsWith(":0:HITONIGHT"));
        System.out.println("Message hash: " + hash);
    }

    @Test
    public void messageHash_incorrect() {
        messages msg = new messages("271234567890", "Hello World", 5);
        String hash = msg.createMessageHash();
        Assertions.assertFalse(hash.endsWith(":0:HITONIGHT"),
            "Hash should not match :0:HITONIGHT but was: " + hash);
        System.out.println("Message hash is incorrect as expected: " + hash);
    }

    @Test
    public void messageHash_loop() {
        String[] recipients   = {"271234567890", "271234567890"};
        String[] texts        = {
            "Hi Mike, can you join us for dinner tonight?",
            "Hi Keegan, did you receive the payment?"
        };
        String[] expectedEnds = {":0:HITONIGHT", ":1:HIPAYMENT"};

        for (int i = 0; i < recipients.length; i++) {
            messages msg = new messages(recipients[i], texts[i], i);
            String hash  = msg.createMessageHash();
            Assertions.assertTrue(hash.endsWith(expectedEnds[i]),
                "Hash failed for message " + (i + 1) + ": " + hash);
            System.out.println("Message " + (i + 1) + " hash: " + hash);
        }
    }

    @Test
    public void messageHash_loop_incorrect() {
        String[] recipients = {"271234567890", "271234567890"};
        String[] texts      = {
            "Hi Mike, can you join us for dinner tonight?",
            "Hi Keegan, did you receive the payment?"
        };
        String[] wrongEnds  = {":0:BYETONIGHT", ":1:HIPAYMENT123"};

        for (int i = 0; i < recipients.length; i++) {
            messages msg = new messages(recipients[i], texts[i], i);
            String hash  = msg.createMessageHash();
            Assertions.assertFalse(hash.endsWith(wrongEnds[i]),
                "Hash should not match " + wrongEnds[i] + " but was: " + hash);
            System.out.println("Message " + (i + 1)
                + " hash incorrectly formatted as expected: " + hash);
        }
    }

    // =========================================================================
    // PART 3 TESTS
    // =========================================================================

    /**
     * Test 1 — Sent Messages array correctly populated.
     * Only msg1 (Sent) and msg4 (Sent) should be in sentMessages.
     */
    @Test
    public void sentMessages_correctlyPopulated() {
        ArrayList<String> expected = new ArrayList<>();
        expected.add("Did you get the cake?");
        expected.add("It is dinner time !");

        ArrayList<String> actual = messages.getSentMessageTexts();

        Assertions.assertEquals(expected, actual,
            "Sent messages array should contain exactly msg1 and msg4 in order.");
        System.out.println("Sent messages array correctly populated: " + actual);
    }

    /**
     * Test 2 — Display the longest Message.
     * Across all 5 test messages, msg2 is the longest.
     */
    @Test
    public void displayLongestMessage_correct() {
        String expected =
            "Where are you? You are late! I have asked you to be on time.";
        String actual = messages.findLongestMessage();

        Assertions.assertEquals(expected, actual,
            "The longest message should be Message 2.");
        System.out.println("Longest message: " + actual);
    }

    /**
     * Test 3 — Search for messageID.
     * msg4 was created with developer-supplied messageID "0838884567".
     */
    @Test
    public void searchByMessageID_found() {
        String expected = "It is dinner time !";
        String actual   = messages.searchMessageByID("0838884567");

        Assertions.assertEquals(expected, actual,
            "Searching for ID '0838884567' should return Message 4's text.");
        System.out.println("Message found by ID '0838884567': " + actual);
    }

    /**
     * Test 4 — Search all messages for a particular recipient.
     * +27838884567 appears on msg2 (Stored) and msg5 (Stored).
     */
    @Test
    public void searchByRecipient_correct() {
        ArrayList<String> expected = new ArrayList<>();
        expected.add("Where are you? You are late! I have asked you to be on time.");
        expected.add("Ok, I am leaving without you.");

        ArrayList<String> actual = messages.getMessagesByRecipient("+27838884567");

        Assertions.assertEquals(expected, actual,
            "Recipient +27838884567 should have exactly two messages (msg2 and msg5).");
        System.out.println("Messages found for +27838884567: " + actual);
    }

    /**
     * Test 5 — Delete a message using a message hash.
     * Retrieves msg2's hash dynamically, then deletes by it.
     */
    @Test
    public void deleteByHash_correct() {
        // msg2 is the first entry added to storedMessages in @BeforeEach
        String hashToDelete = messages.getStoredMessages().get(0).getMessageHash();

        String expected = "Message: \""
            + "Where are you? You are late! I have asked you to be on time."
            + "\" successfully deleted.";
        String actual = messages.deleteMessageByHash(hashToDelete);

        Assertions.assertEquals(expected, actual,
            "Deleting msg2 by its hash should return the correct confirmation.");
        System.out.println("Delete result: " + actual);
    }

    /**
     * Test 6 — Display Report.
     * Report must list all sent messages including hash, recipient, and message.
     */
    @Test
    public void displayReport_containsAllSentMessages() {
        String report = messages.displayReport();

        Assertions.assertTrue(report.contains("Message Hash:"),
            "Report must include 'Message Hash:' labels.");
        Assertions.assertTrue(report.contains("Recipient:"),
            "Report must include 'Recipient:' labels.");
        Assertions.assertTrue(report.contains("Message:"),
            "Report must include 'Message:' labels.");
        Assertions.assertTrue(report.contains("Did you get the cake?"),
            "Report must contain Message 1 text.");
        Assertions.assertTrue(report.contains("It is dinner time !"),
            "Report must contain Message 4 text.");

        System.out.println("Display report test passed.\n" + report);
    }
}
