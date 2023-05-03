# Online-Voting-System-Code
Blockchain Project Code
package block;
import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.control.RadioButton;
import javafx.scene.control.ToggleGroup;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;
import org.web3j.abi.datatypes.generated.Uint256;

public class EVotingSystem extends Application {

    private static final String CONTRACT_ADDRESS = "0x456..."; // replace with your contract address

    private final VotingContract contract = VotingContract.load(CONTRACT_ADDRESS, web3j, credentials, GAS_PRICE, GAS_LIMIT);

    private final ToggleGroup candidateGroup = new ToggleGroup();

    @Override
    public void start(Stage stage) {
        Label heading = new Label("E-Voting System");
        heading.setStyle("-fx-font-size: 24pt; -fx-font-weight: bold;");

        Label label = new Label("Please select a candidate:");

        RadioButton candidate1 = new RadioButton("John Smith");
        candidate1.setToggleGroup(candidateGroup);

        RadioButton candidate2 = new RadioButton("Jane Doe");
        candidate2.setToggleGroup(candidateGroup);

        Button voteButton = new Button("Vote");
        voteButton.setOnAction(event -> {
            int candidateIndex = Integer.parseInt(candidateGroup.getSelectedToggle().getUserData().toString());
            try {
                contract.castVote(new Uint256(candidateIndex)).send();
                System.out.println("Your vote has been recorded.");
            } catch (Exception e) {
                e.printStackTrace();
            }
        });

        VBox vbox = new VBox(10, heading, label, candidate1, candidate2, voteButton);
        vbox.setPadding(new Insets(20));
        vbox.setStyle("-fx-background-color: #F5F5F5;");

        Scene scene = new Scene(vbox, 400, 300);
        stage.setTitle("E-Voting System");
        stage.setScene(scene);
        stage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }

}
import org.web3j.abi.datatypes.Address;
import org.web3j.abi.datatypes.Bool;
import org.web3j.abi.datatypes.generated.Uint256;
import org.web3j.crypto.Credentials;
import org.web3j.protocol.Web3j;
import org.web3j.protocol.core.DefaultBlockParameterName;
import org.web3j.protocol.core.methods.response.TransactionReceipt;
import org.web3j.protocol.http.HttpService;
import org.web3j.tx.gas.DefaultGasProvider;

import java.math.BigInteger;

public class EVotingSystemBackend {

    private final Web3j web3j;
    private final Credentials credentials;
    private final EVotingSystem contract;
    private final String contractAddress;

    public EVotingSystemBackend(String rpcUrl, String privateKey, String contractAddress) {
        this.web3j = Web3j.build(new HttpService(rpcUrl));
        this.credentials = Credentials.create(privateKey);
        this.contractAddress = contractAddress;
        this.contract = EVotingSystem.load(
            contractAddress, web3j, credentials, new DefaultGasProvider()
        );
    }

    public void addCandidate(String name) throws Exception {
        TransactionReceipt tx = contract.addCandidate(name).send();
        System.out.println("Add candidate transaction hash: " + tx.getTransactionHash());
    }

    public void openVoting() throws Exception {
        TransactionReceipt tx = contract.openVoting().send();
        System.out.println("Open voting transaction hash: " + tx.getTransactionHash());
    }

    public void closeVoting() throws Exception {
        TransactionReceipt tx = contract.closeVoting().send();
        System.out.println("Close voting transaction hash: " + tx.getTransactionHash());
    }

    public void castVote(int candidateIndex) throws Exception {
        TransactionReceipt tx = contract.castVote(new Uint256(candidateIndex)).send();
        System.out.println("Cast vote transaction hash: " + tx.getTransactionHash());
    }

    public String getWinner() throws Exception {
        Address winnerAddress = contract.getWinner().send();
        return winnerAddress.getValue();
    }

    public BigInteger getTotalVotes() throws Exception {
        Uint256 totalVotes = contract.totalVotes().send();
        return totalVotes.getValue();
    }

    public boolean isVotingOpen() throws Exception {
        Bool votingOpen = contract.votingOpen().send();
        return votingOpen.getValue();
    }

    public static void main(String[] args) throws Exception {
        String rpcUrl = "http://localhost:8545"; // replace with your RPC endpoint
        String privateKey = "0x123..."; // replace with your private key
        String contractAddress = "0x456..."; // replace with your contract address

        EVotingSystemBackend backend = new EVotingSystemBackend(rpcUrl, privateKey, contractAddress);

        backend.addCandidate("John Smith");
        backend.addCandidate("Jane Doe");
        backend.openVoting();
        backend.castVote(0);
        backend.castVote(1);
        backend.closeVoting();

        System.out.println("Winner: " + backend.getWinner());
        System.out.println("Total votes: " + backend.getTotalVotes());
        System.out.println("Voting open: " + backend.isVotingOpen());
    }
}
