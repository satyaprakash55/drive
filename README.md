Catalog Placements Assignment - Online

Explanation
1. Reading Input
The readFile() method reads JSON files from disk

Two test cases are read: testcase1.json and testcase2.json

2. JSON Parsing
parseJsonInput() handles the custom JSON parsing:

Removes JSON syntax characters

Extracts the number of points (n)

For each point, extracts the x value (the key), base, and encoded y value

Decodes y values using BigInteger(value, base)

3. Polynomial Reconstruction
getKValue() extracts the minimum number of points needed (k)

lagrangeInterpolate() implements Lagrange interpolation:

Constructs the polynomial from k points

Evaluates it at x=0 to get the constant term (secret)

Uses BigInteger for arbitrary-precision arithmetic

4. Key Features
No external dependencies - Custom JSON parsing

Handles large numbers - Uses BigInteger

Base conversion - Supports bases 2-36

Error handling - Basic validation

How to Use
Create two JSON files:

testcase1.json with the first test case

testcase2.json with the second test case.    



                           " CODES " 



import java.math.BigInteger;
import java.util.*;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

public class ShamirSecretDecoder {

    static class Point {
        final BigInteger x;
        final BigInteger y;
        
        Point(BigInteger x, BigInteger y) {
            this.x = x;
            this.y = y;
        }
    }

    public static void main(String[] args) {
        try {
            // Read and process both test cases
            String testCase1 = readFile("testcase1.json");
            String testCase2 = readFile("testcase2.json");
            
            System.out.println("Secret for Test Case 1: " + processTestCase(testCase1));
            System.out.println("Secret for Test Case 2: " + processTestCase(testCase2));
            
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    private static String readFile(String filename) throws IOException {
        return new String(Files.readAllBytes(Paths.get(filename)));
    }

    private static BigInteger processTestCase(String jsonInput) throws Exception {
        // Parse JSON and extract points
        List<Point> points = parseJsonInput(jsonInput);
        
        // Get k value from input (minimum points needed)
        int k = getKValue(jsonInput);
        
        // Use Lagrange interpolation to find the constant term
        return lagrangeInterpolate(points, k, BigInteger.ZERO);
    }

    private static List<Point> parseJsonInput(String json) throws Exception {
        List<Point> points = new ArrayList<>();
        json = json.replaceAll("[{}\"\\s]", ""); // Remove JSON syntax
        
        // First extract keys.n and keys.k
        int n = 0;
        String[] pairs = json.split(",");
        for (String pair : pairs) {
            if (pair.startsWith("keys.n:")) {
                n = Integer.parseInt(pair.split(":")[1]);
                break;
            }
        }
        
        // Then extract all the points
        for (int i = 1; i <= n; i++) {
            String x = String.valueOf(i);
            String baseKey = x + ".base:";
            String valueKey = x + ".value:";
            
            int base = 0;
            String value = null;
            
            for (String pair : pairs) {
                if (pair.startsWith(baseKey)) {
                    base = Integer.parseInt(pair.substring(baseKey.length()));
                } else if (pair.startsWith(valueKey)) {
                    value = pair.substring(valueKey.length());
                }
            }
            
            if (base != 0 && value != null) {
                points.add(new Point(
                    new BigInteger(x),
                    new BigInteger(value, base)
                ));
            }
        }
        
        return points;
    }

    private static int getKValue(String json) {
        json = json.replaceAll("[{}\"\\s]", "");
        String[] pairs = json.split(",");
        for (String pair : pairs) {
            if (pair.startsWith("keys.k:")) {
                return Integer.parseInt(pair.split(":")[1]);
            }
        }
        throw new IllegalArgumentException("k value not found in JSON");
    }

    private static BigInteger lagrangeInterpolate(List<Point> points, int k, BigInteger x) {
        BigInteger secret = BigInteger.ZERO;
        
        for (int i = 0; i < k; i++) {
            BigInteger numerator = points.get(i).y;
            BigInteger denominator = BigInteger.ONE;
            
            for (int j = 0; j < k; j++) {
                if (i != j) {
                    numerator = numerator.multiply(x.subtract(points.get(j).x));
                    denominator = denominator.multiply(
                        points.get(i).x.subtract(points.get(j).x)
                    );
                }
            }
            
            secret = secret.add(numerator.divide(denominator));
        }
        
        return secret;
    }
}

