# Scientific-Calculator
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.lang.Math;

public class ScientificCalculator extends JFrame implements ActionListener {
    JTextField display;
    StringBuilder input;

    public ScientificCalculator() {
        input = new StringBuilder();
        setTitle("Scientific Calculator");
        setSize(400, 500);
        setLayout(new BorderLayout());
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        display = new JTextField();
        display.setEditable(false);
        display.setFont(new Font("Arial", Font.PLAIN, 24));
        add(display, BorderLayout.NORTH);

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(6, 4, 5, 5));

        String[] buttons = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", ".", "=", "+",
            "sin", "cos", "tan", "sqrt",
            "log", "pow", "C", "DEL"
        };

        for (String text : buttons) {
            JButton button = new JButton(text);
            button.setFont(new Font("Arial", Font.PLAIN, 18));
            button.addActionListener(this);
            panel.add(button);
        }

        add(panel, BorderLayout.CENTER);
        setVisible(true);
    }

    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();

        try {
            switch (command) {
                case "C":
                    input.setLength(0);
                    break;
                case "DEL":
                    if (input.length() > 0) {
                        input.setLength(input.length() - 1);
                    }
                    break;
                case "=":
                    display.setText(evaluate(input.toString()));
                    input.setLength(0);
                    break;
                case "sin":
                case "cos":
                case "tan":
                case "log":
                case "sqrt":
                case "pow":
                    input.append(command).append("(");
                    break;
                default:
                    input.append(command);
            }
            display.setText(input.toString());
        } catch (Exception ex) {
            display.setText("Error");
            input.setLength(0);
        }
    }

    // Evaluate mathematical expression
    private String evaluate(String expr) {
        expr = expr.replaceAll("sin\\(", "Math.sin(Math.toRadians(")
                   .replaceAll("cos\\(", "Math.cos(Math.toRadians(")
                   .replaceAll("tan\\(", "Math.tan(Math.toRadians(")
                   .replaceAll("log\\(", "Math.log10(")
                   .replaceAll("sqrt\\(", "Math.sqrt(")
                   .replaceAll("pow\\(", "Math.pow(");
        try {
            Object result = new javax.script.ScriptEngineManager()
                                .getEngineByName("JavaScript")
                                .eval(expr);
            return result.toString();
        } catch (Exception e) {
            return "Invalid";
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new ScientificCalculator());
    }
}
