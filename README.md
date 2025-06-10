#calculatorimport java.util.*;

public class calWithPrecedence {

    public static int getPrecedence(String op) {
        switch (op) {
            case "+": case "-":
                return 1;
            case "*": case "/":
                return 2;
            default:
                return 0;
        }
    }

    public static boolean isOperator(String token) {
        return token.matches("[+\\-*/]");
    }

    public static boolean isNumber(String token) {
        return token.matches("-?\\d+(\\.\\d+)?");
    }

    public static LinkedList<String> infixToPostfix(String expr) {
        LinkedList<String> output = new LinkedList<>();
        LinkedList<String> stack = new LinkedList<>();

        StringTokenizer tokenizer = new StringTokenizer(expr, "+-*/() ", true);
        while (tokenizer.hasMoreTokens()) {
            String token = tokenizer.nextToken().trim();
            if (token.isEmpty()) continue;

            if (isNumber(token)) {
                output.add(token);
            } else if (isOperator(token)) {
                while (!stack.isEmpty() && isOperator(stack.peekFirst())
                        && getPrecedence(token) <= getPrecedence(stack.peekFirst())) {
                    output.add(stack.removeFirst());
                }
                stack.addFirst(token);
            } else if (token.equals("(")) {
                stack.addFirst(token);
            } else if (token.equals(")")) {
                while (!stack.isEmpty() && !stack.peekFirst().equals("(")) {
                    output.add(stack.removeFirst());
                }
                if (!stack.isEmpty() && stack.peekFirst().equals("(")) {
                    stack.removeFirst(); // remove '('
                } else {
                    System.out.println("Error: Mismatched parentheses");
                    return null;
                }
            } else {
                System.out.println("Invalid token: " + token);
                return null;
            }
        }

        while (!stack.isEmpty()) {
            if (stack.peekFirst().equals("(")) {
                System.out.println("Error: Mismatched parentheses");
                return null;
            }
            output.add(stack.removeFirst());
        }

        return output;
    }

    public static double evaluatePostfix(LinkedList<String> postfix, LinkedList<Number> enteredNumbers) {
        LinkedList<Double> stack = new LinkedList<>();

        for (String token : postfix) {
            if (isNumber(token)) {
                double val = Double.parseDouble(token);
                stack.addFirst(val);

                if (val == (int) val)
                    enteredNumbers.add((int) val);
                else
                    enteredNumbers.add(val);
            } else if (isOperator(token)) {
                double b = stack.removeFirst();
                double a = stack.removeFirst();
                double res = 0;
                switch (token) {
                    case "+": res = a + b; break;
                    case "-": res = a - b; break;
                    case "*": res = a * b; break;
                    case "/":
                        if (b == 0) throw new ArithmeticException("Division by zero");
                        res = a / b; break;
                }
                stack.addFirst(res);
            }
        }
        return stack.removeFirst();
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        while (true) {
            LinkedList<Number> enteredNumbers = new LinkedList<>();
            System.out.print("Enter expression with operators and parentheses: ");
            String expr = sc.nextLine();

            LinkedList<String> postfix = infixToPostfix(expr);
            if (postfix == null) continue;

            try {
                double result = evaluatePostfix(postfix, enteredNumbers);
                System.out.println("Final result: " + result);
                System.out.println("Numbers entered (unsorted): " + enteredNumbers);

                LinkedList<Number> sorted = new LinkedList<>(enteredNumbers);
                sorted.sort(Comparator.comparingDouble(Number::doubleValue));
                System.out.println("Numbers entered (sorted): " + sorted);

                LinkedList<Number> even = new LinkedList<>();
                LinkedList<Number> odd = new LinkedList<>();
                for (Number n : sorted) {
                    if (n instanceof Integer) {
                        if ((Integer) n % 2 == 0) even.add(n);
                        else odd.add(n);
                    } else {
                        odd.add(n);
                    }
                }

                System.out.println("Even numbers: " + even);
                System.out.println("Odd numbers: " + odd);
            } catch (Exception e) {
                System.out.println("Error: " + e.getMessage());
            }

            System.out.print("Do you want to calculate another expression? (y/n): ");
            if (!sc.nextLine().equalsIgnoreCase("y")) {
                System.out.println("Thank you for using the calculator!");
                break;
            }
        }

        sc.close();
    }
}
