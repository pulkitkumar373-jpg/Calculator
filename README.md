import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class ProfessionalCalc extends JFrame implements ActionListener {
    private JTextField screen;
    private JLabel operatorLabel;
    private DefaultListModel<String> historyModel;
    private JList<String> historyList;
    private JScrollPane scrollPane;
    
    private double firstNum = 0;
    private String currentOperator = "";
    private boolean startNewNumber = true;

    public ProfessionalCalc() {
        setTitle("Saraswati Pro Calc 2.0");
        setSize(400, 600); 
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout(10, 10));
        getContentPane().setBackground(Color.WHITE);

        // --- TOP PANEL ---
        JPanel topPanel = new JPanel(new BorderLayout());
        topPanel.setBackground(Color.WHITE);
        
        operatorLabel = new JLabel(" ", SwingConstants.RIGHT);
        operatorLabel.setFont(new Font("SansSerif", Font.BOLD, 18));
        operatorLabel.setForeground(new Color(0, 122, 255));
        operatorLabel.setBorder(BorderFactory.createEmptyBorder(10, 20, 0, 20));
        
        screen = new JTextField("0");
        screen.setEditable(false);
        screen.setFont(new Font("SansSerif", Font.PLAIN, 45));
        screen.setBackground(Color.WHITE);
        screen.setForeground(new Color(50, 50, 50));
        screen.setHorizontalAlignment(JTextField.RIGHT);
        screen.setBorder(BorderFactory.createEmptyBorder(0, 20, 20, 20));
        
        topPanel.add(operatorLabel, BorderLayout.NORTH);
        topPanel.add(screen, BorderLayout.CENTER);
        add(topPanel, BorderLayout.NORTH);

        // --- CENTER PANEL (Buttons) ---
        JPanel btnPanel = new JPanel(new GridLayout(5, 4, 8, 8)); 
        btnPanel.setBackground(Color.WHITE);
        btnPanel.setBorder(BorderFactory.createEmptyBorder(0, 10, 10, 10));

        String[] buttons = {
            "HIST", "C", "DEL", "/",
            "7", "8", "9", "*",
            "4", "5", "6", "-",
            "1", "2", "3", "+",
            "±", "0", ".", "="
        };

        for (String text : buttons) {
            JButton b = new JButton(text);
            styleButton(b, text);
            b.addActionListener(this);
            btnPanel.add(b);
        }
        add(btnPanel, BorderLayout.CENTER);

        // --- RIGHT PANEL (History) ---
        historyModel = new DefaultListModel<>();
        historyList = new JList<>(historyModel);
        historyList.setFont(new Font("SansSerif", Font.PLAIN, 12));
        scrollPane = new JScrollPane(historyList);
        scrollPane.setPreferredSize(new Dimension(180, 0));
        scrollPane.setBorder(BorderFactory.createTitledBorder("History"));
        scrollPane.setVisible(false);
        add(scrollPane, BorderLayout.EAST);

        setLocationRelativeTo(null);
        setVisible(true);
    }

    private void styleButton(JButton b, String text) {
        b.setFont(new Font("Segoe UI", Font.BOLD, 16));
        b.setFocusPainted(false);
        b.setBorder(BorderFactory.createLineBorder(new Color(240, 240, 240)));
        if (text.equals("=")) {
            b.setBackground(new Color(0, 122, 255));
            b.setForeground(Color.WHITE);
        } else {
            b.setBackground(new Color(252, 252, 252));
            b.setForeground(new Color(70, 70, 70));
        }
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String cmd = e.getActionCommand();

        if (cmd.equals("HIST")) {
            scrollPane.setVisible(!scrollPane.isVisible());
            if (scrollPane.isVisible()) setSize(580, 600);
            else setSize(400, 600);
            revalidate();
        } 
        else if ("0123456789.".contains(cmd)) {
            if (startNewNumber) { screen.setText(cmd); startNewNumber = false; }
            else { 
                if (cmd.equals(".") && screen.getText().contains(".")) return;
                screen.setText(screen.getText() + cmd); 
            }
        } 
        else if (cmd.equals("C")) {
            screen.setText("0"); operatorLabel.setText(" ");
            firstNum = 0; currentOperator = ""; startNewNumber = true;
        } 
        else if (cmd.equals("DEL")) {
            String val = screen.getText();
            if (val.length() > 1) screen.setText(val.substring(0, val.length() - 1));
            else { screen.setText("0"); startNewNumber = true; }
        } 
        else if (cmd.equals("±")) {
            double val = Double.parseDouble(screen.getText()) * -1;
            screen.setText(formatResult(val));
        }
        else if ("+-*/%".contains(cmd)) {
            firstNum = Double.parseDouble(screen.getText());
            currentOperator = cmd;
            operatorLabel.setText(formatResult(firstNum) + " " + cmd);
            startNewNumber = true;
        } 
        else if (cmd.equals("=")) {
            if (currentOperator.isEmpty()) return;
            double secondNum = Double.parseDouble(screen.getText());
            double result = 0;
            
            switch (currentOperator) {
                case "+": result = firstNum + secondNum; break;
                case "-": result = firstNum - secondNum; break;
                case "*": result = firstNum * secondNum; break;
                case "/": result = secondNum != 0 ? firstNum / secondNum : 0; break;
                case "%": result = firstNum % secondNum; break;
            }
            
            String resStr = formatResult(result);
            // UPDATE: Yahan '=' ko label mein add kiya hai
            operatorLabel.setText(formatResult(firstNum) + " " + currentOperator + " " + formatResult(secondNum) + " =");
            
            historyModel.addElement(formatResult(firstNum) + " " + currentOperator + " " + formatResult(secondNum) + " = " + resStr);
            screen.setText(resStr);
            currentOperator = "";
            startNewNumber = true;
        }
    }

    private String formatResult(double d) {
        if (d == (long) d) return String.format("%d", (long) d);
        else return String.valueOf(d);
    }

    public static void main(String[] args) {
        try { UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName()); } catch (Exception e) {}
        new ProfessionalCalc();
    }
}
