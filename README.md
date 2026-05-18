public String checkReceiverCell(){
    if (receiver != null && receiver.matches("^27[0-9]{10}$")){
        return "Cell number validated.";  // ← was "cell number is valid"
    }
    return "Cell number is incorrect. Please enter the correct number";
}

