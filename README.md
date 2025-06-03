import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import java.text.DecimalFormat;

public class ScientificCalculator extends JFrame implements ActionListener {
    private JTextArea display;
    private StringBuilder input;

    public ScientificCalculator() {
        input = new StringBuilder();
        setTitle("Scientific Calculator");
        setSize(500, 700);
        setLayout(new BorderLayout(10, 10));
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        getContentPane().setBackground(new Color(245, 245, 245));

        // === Display ===
        display = new JTextArea(3, 30);
        display.setFont(new Font("Consolas", Font.BOLD, 26));
        display.setEditable(false);
        display.setLineWrap(true);
        display.setWrapStyleWord(true);
        display.setBackground(Color.WHITE);
        display.setForeground(Color.BLACK);
        display.setBorder(BorderFactory.createLineBorder(Color.LIGHT_GRAY, 2));
        JScrollPane scroll = new JScrollPane(display);
        add(scroll, BorderLayout.NORTH);

        // === Buttons ===
        String[] buttons = {
            "(", ")", ".", "%",
            "7", "8", "9", "+",
            "4", "5", "6", "-",
            "1", "2", "3", "*",
            "π", "0", "C", "/",
            "sin", "cos", "tan", "!",
            "log", "ln", "sqrt", "^",
             " " ,"Enter", "DEL","",
        };

        JPanel panel = new JPanel(new GridLayout(8, 4, 10, 10));
        panel.setBackground(new Color(245, 245, 245));

        for (String text : buttons) {
            JButton button = new JButton(text);
            button.setFont(new Font("Arial", Font.BOLD, 18));
            styleButton(button, text);
            button.addActionListener(this);
            panel.add(button);
        }

        add(panel, BorderLayout.CENTER);
        addKeyBindings();
        setVisible(true);
        display.requestFocusInWindow();
    }

    private void styleButton(JButton button, String text) {
        button.setFocusPainted(false);
        button.setOpaque(true);
        button.setBorder(BorderFactory.createLineBorder(Color.GRAY, 1));
        button.setForeground(Color.BLACK);

        Color white = Color.WHITE;
        Color lightGrey = new Color(230, 230, 230);
        Color darkYellow = new Color(255, 235, 59); // darker yellow

        if ("CDELπ".contains(text)) {
            button.setBackground(lightGrey); // Control buttons
        } else if ("+-*/%^!".contains(text)) {
            button.setBackground(darkYellow); // Arithmetic ops
        } else if ("sin cos tan log ln sqrt".contains(text)) {
            button.setBackground(new Color(240, 240, 240)); // Functions
        } else if (text.equals("Enter")) {
            button.setBackground(new Color(220, 220, 220)); // Enter key
        } else {
            button.setBackground(white); // Digits and brackets
        }
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        handleInput(e.getActionCommand());
    }

    private void handleInput(String cmd) {
        try {
            switch (cmd) {
                case "C" -> input.setLength(0);
                case "DEL" -> {
                    if (input.length() > 0) input.setLength(input.length() - 1);
                }
                case "Enter" -> {
                    String result = evaluate(input.toString());
                    display.setText(result);
                    input.setLength(0);
                    return;
                }
                case "π" -> input.append(Math.PI);
                case "sin", "cos", "tan", "log", "ln", "sqrt" -> input.append(cmd).append("(");
                default -> input.append(cmd);
            }
            display.setText(input.toString());
        } catch (Exception ex) {
            display.setText("Error");
            input.setLength(0);
        }
    }

    private String evaluate(String expr) {
        try {
            expr = expr.replace("π", String.valueOf(Math.PI));
            expr = expr.replaceAll("sin\\(([^)]+)\\)", "Math.sin(($1)*Math.PI/180)");
            expr = expr.replaceAll("cos\\(([^)]+)\\)", "Math.cos(($1)*Math.PI/180)");
            expr = expr.replaceAll("tan\\(([^)]+)\\)", "Math.tan(($1)*Math.PI/180)");
            expr = expr.replaceAll("log\\(([^)]+)\\)", "Math.log10($1)");
            expr = expr.replaceAll("ln\\(([^)]+)\\)", "Math.log($1)");
            expr = expr.replaceAll("sqrt\\(([^)]+)\\)", "Math.sqrt($1)");
            expr = expr.replaceAll("(\\d+(?:\\.\\d+)?)(\\^)(\\d+(?:\\.\\d+)?)", "Math.pow($1,$3)");
            expr = expr.replaceAll("(\\d+)!","factorial($1)");

            // Evaluate expression using JavaScript engine
            ScriptEngine engine = new ScriptEngineManager().getEngineByName("JavaScript");

            // Define factorial function
            engine.eval("function factorial(n) { if (n==0||n==1) return 1; var r=1; for (var i=2;i<=n;i++) r*=i; return r; }");

            Object result = engine.eval(expr);
            double value = Double.parseDouble(result.toString());
            DecimalFormat df = new DecimalFormat("0.#####");
            return df.format(value);
        } catch (Exception e) {
            return "Invalid Expression";
        }
    }

    private void addKeyBindings() {
        JComponent root = getRootPane();
        InputMap im = root.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW);
        ActionMap am = root.getActionMap();

        String keys = "0123456789+-*/().^!";

        for (char ch : keys.toCharArray()) {
            im.put(KeyStroke.getKeyStroke(ch), "append_" + ch);
            am.put("append_" + ch, new AbstractAction() {
                public void actionPerformed(ActionEvent e) {
                    input.append(ch);
                    display.setText(input.toString());
                }
            });
        }

        im.put(KeyStroke.getKeyStroke(KeyEvent.VK_BACK_SPACE, 0), "delete");
        am.put("delete", new AbstractAction() {
            public void actionPerformed(ActionEvent e) {
                if (input.length() > 0) input.setLength(input.length() - 1);
                display.setText(input.toString());
            }
        });

        im.put(KeyStroke.getKeyStroke(KeyEvent.VK_ENTER, 0), "evaluate");
        am.put("evaluate", new AbstractAction() {
            public void actionPerformed(ActionEvent e) {
                String result = evaluate(input.toString());
                display.setText(result);
                input.setLength(0);
            }
        });
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(ScientificCalculator::new);
    }
}
