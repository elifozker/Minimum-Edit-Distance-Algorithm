package nlp;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;


public class MedAlgorithm extends JFrame {
	 	private JTextField inputWordField;
	    private JButton calculateButton;
	    private JTextArea resultArea;

	    private JTextField inputWordField2;
	    private JTextField inputWordField3;
	    private JButton calculateButton2;
	    private JTextArea resultArea2;

	    public MedAlgorithm() {
	        setTitle("NLP MED ALGORITHM GUI");
	        setSize(1800, 1600);
	        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
	        setLocationRelativeTo(null);

	        // PART-1
	        inputWordField = new JTextField(20);
	        calculateButton = new JButton("Calculate Distances for Part-1");
	        resultArea = new JTextArea(15, 20);
	        resultArea.setEditable(false);

	        JPanel inputPanel = new JPanel(new FlowLayout());
	        inputPanel.add(new JLabel("Enter a word: "));
	        inputPanel.add(inputWordField);
	        inputPanel.add(calculateButton);

	        JPanel resultPanel1 = new JPanel(new BorderLayout());
	        resultPanel1.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
	        resultPanel1.add(new JLabel("PART - 1"), BorderLayout.NORTH); 
	        resultPanel1.add(new JScrollPane(resultArea), BorderLayout.CENTER);

	        // PART-2
	        inputWordField2 = new JTextField(10);
	        inputWordField3 = new JTextField(10);
	        calculateButton2 = new JButton("Calculate Distances for Part-2");
	        resultArea2 = new JTextArea(35, 40);
	        resultArea2.setEditable(false);

	        JPanel inputPanel2 = new JPanel(new FlowLayout());
	        inputPanel2.add(new JLabel("Enter a word1: "));
	        inputPanel2.add(inputWordField2);
	        inputPanel2.add(new JLabel("Enter a word2: "));
	        inputPanel2.add(inputWordField3);
	        inputPanel2.add(calculateButton2);

	        JPanel resultPanel2 = new JPanel(new BorderLayout());
	        resultPanel2.setBorder(BorderFactory.createEmptyBorder(5, 5, 5, 5));
	        resultPanel2.add(new JLabel("PART - 2"), BorderLayout.NORTH); 
	        resultPanel2.add(new JScrollPane(resultArea2), BorderLayout.CENTER);

	        JSplitPane splitPane = new JSplitPane(JSplitPane.VERTICAL_SPLIT, inputPanel, inputPanel2);
	        splitPane.setDividerLocation(100); 
	        splitPane.setEnabled(false); 

	        add(splitPane, BorderLayout.CENTER);
	        add(resultPanel1, BorderLayout.BEFORE_LINE_BEGINS);
	        add(resultPanel2, BorderLayout.PAGE_END);

	        calculateButton.addActionListener(new ActionListener() {
	            @Override
	            public void actionPerformed(ActionEvent e) {
	                String inputWord = inputWordField.getText().toLowerCase();
	                long startTime = System.nanoTime();
	                calculateAndDisplayDistances(inputWord);
	                long endTime   = System.nanoTime();
	                double seconds = (double) (endTime - startTime) / 1000000000;
	                System.out.println(seconds);

	                
	            }
	        });

	        calculateButton2.addActionListener(new ActionListener() {
	            @Override
	            public void actionPerformed(ActionEvent e) {
	                String inputWord1 = inputWordField2.getText().toLowerCase();
	                String inputWord2 = inputWordField3.getText().toLowerCase();
	                long startTime = System.nanoTime();
	                displayCostMatrix(inputWord1, inputWord2);
	                long endTime   = System.nanoTime();
	                double seconds = (double) (endTime - startTime) / 1000000000;
	                System.out.println(seconds);
	            }
	        });
	    }

 
    private void calculateAndDisplayDistances(String inputWord) {
        List<String> wordList = readWordListFromFile("/Users/elifozker/Desktop/vocabulary_tr.txt");
        Map<String, Integer> wordDistances = calculateDistances(inputWord, wordList);
        displayTopFiveWords(wordDistances, 5);
    }
    private List<Map<String, String>> transformWords(int[][] costMatrix, String word1, String word2) {
        List<Map<String, String>> operations = new ArrayList<>();
        int m = word1.length();
        int n = word2.length();
        int indexm = m;
        int indexn = n;

        while (indexm > 0 || indexn > 0) {
            Map<String, String> operationDetails = new HashMap<>();

            if (indexm == 0) {
                operationDetails.put("operation", "Insert");
                operationDetails.put("character", String.valueOf(word2.charAt(indexn - 1)));
                operations.add(operationDetails);
                indexn--;
            } else if (indexn == 0) {
                operationDetails.put("operation", "Delete");
                operationDetails.put("character", String.valueOf(word1.charAt(indexm - 1)));
                operations.add(operationDetails);
                indexm--;
            } else {
                if (word1.charAt(indexm - 1) == word2.charAt(indexn - 1)) {
                    operationDetails.put("operation", "Same word");
                    operationDetails.put("character", String.valueOf(word2.charAt(indexn - 1)));
                    operations.add(operationDetails);
                    indexm--;
                    indexn--;
                } else if (costMatrix[indexm][indexn - 1] < costMatrix[indexm - 1][indexn] && costMatrix[indexm][indexn - 1] < costMatrix[indexm - 1][indexn - 1]) {
                    operationDetails.put("operation", "Insert");
                    operationDetails.put("character", String.valueOf(word2.charAt(indexn - 1)));
                    operations.add(operationDetails);
                    indexn--;
                } else if (costMatrix[indexm - 1][indexn] < costMatrix[indexm][indexn - 1] && costMatrix[indexm - 1][indexn] < costMatrix[indexm - 1][indexn - 1]) {
                    operationDetails.put("operation", "Delete");
                    operationDetails.put("character", String.valueOf(word1.charAt(indexm - 1)));
                    operations.add(operationDetails);
                    indexm--;
                } else {
                    operationDetails.put("operation", "Replace");
                    operationDetails.put("oldCharacter", String.valueOf(word1.charAt(indexm - 1)));
                    operationDetails.put("newCharacter", String.valueOf(word2.charAt(indexn - 1)));
                    operationDetails.put("position", String.valueOf(indexm));
                    operations.add(operationDetails);
                    indexm--;
                    indexn--;
                }
            }
        }

        Collections.reverse(operations);
        return operations;
    }



    private void displayTopFiveWords(Map<String, Integer> wordDistances, int k) {
    	resultArea.setText("");
        resultArea.append("Top " + k + " words:\n");
        List<Map.Entry<String, Integer>> entryList = new ArrayList<>(wordDistances.entrySet());
        entryList.sort(Map.Entry.comparingByValue());

        Map<String, Integer> sortedMap = new LinkedHashMap<>();
        for (Map.Entry<String, Integer> entry : entryList) {
            sortedMap.put(entry.getKey(), entry.getValue());
        }

        int count = 0;
        for (Map.Entry<String, Integer> entry : sortedMap.entrySet()) {
            resultArea.append(entry.getKey() + ": " + entry.getValue() + "\n");
            count++;
            if (count == k) {
                break;
            }
        }
        resultArea.append("\n");
        
    }

    private void displayCostMatrix(String word1, String word2) {
    	resultArea2.setText("");
        resultArea2.append("Cost Matrix:\n");
        int[][] costMatrix = minEditDistance(word1, word2);

        for (int i = 0; i < costMatrix.length; i++) {
            for (int j = 0; j < costMatrix[0].length; j++) {
                resultArea2.append(costMatrix[i][j] + "\t");
            }
            resultArea2.append("\n");
        }
        List<Map<String, String>> operations = transformWords(costMatrix, word1, word2);
        for (Map<String, String> operationDetails : operations) {
            for (Map.Entry<String, String> entry : operationDetails.entrySet()) {
                String key = entry.getKey();
                String value = entry.getValue();
                resultArea2.append(key + " --- " + value + "\n");  
            }
            resultArea2.append("-----------\n");
        }

       
    }

    private int[][] minEditDistance(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        int[][] costMatrix = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++)
            costMatrix[i][0] = i;
        for (int i = 1; i <= n; i++)
            costMatrix[0][i] = i;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (word1.charAt(i) == word2.charAt(j)) // if a == b
                    costMatrix[i + 1][j + 1] = costMatrix[i][j];
                else {
                    int a = costMatrix[i][j];
                    int b = costMatrix[i][j + 1];
                    int c = costMatrix[i + 1][j];

                    int mina = a + 1;
                    int minb = b + 1;
                    int minc = c + 1;
                    int minValueOfThreeOfThem = Math.min(mina, Math.min(minb, minc));
                    costMatrix[i + 1][j + 1] = minValueOfThreeOfThem;
                }
            }
        }
        return costMatrix;
    }

    private Map<String, Integer> calculateDistances(String inputWord, List<String> wordList) {
        Map<String, Integer> wordDistances = new HashMap<>();

        for (String word : wordList) {
            int[][] costMatrix = minEditDistance(inputWord, word);
            int m = inputWord.length();
            int n = word.length();
            int distance = costMatrix[m][n];
            wordDistances.put(word, distance);
        }

        return wordDistances;
    }

    private List<String> readWordListFromFile(String filePath) {
        List<String> wordList = new ArrayList<>();

        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(new FileInputStream(filePath), StandardCharsets.UTF_8))) {

            String line;
            while ((line = reader.readLine()) != null) {
                wordList.add(line.trim());
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
        return wordList;
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new MedAlgorithm().setVisible(true);
            }
        });
    }
}
