What this project does
You are given several points 
(
x
,
y
)
(x,y) that lie on an unknown polynomial 
P
(
x
)
P(x). Each point is encoded in a small JSON format where the index key is the 
x
x-coordinate (like "1", "2") and the value is a number encoded in an arbitrary base (like base 2, base 10, base 16, etc.). The job is to reconstruct the polynomial (or at least its constant term 
P
(
0
)
P(0)) from those points.
This repo includes:
a robust JS/Node solver that decodes base-encoded numbers (up to base-62), does exact arithmetic with BigInt, computes the constant term using Lagrange interpolation, and verifies the result against all given points, and
a React component PolynomialSolverAnalyzer that provides a UI to run tests and paste custom JSON.
Problem statement (beginner-friendly)
Imagine you know the value of an unknown polynomial at several different 
x
x-positions:
Example: you know
P
(
1
)
=
4
P(1)=4,
P
(
2
)
=
7
P(2)=7,
P
(
3
)
=
12
P(3)=12.
From such data you can find the coefficients of a polynomial that fits those points. If the polynomial has degree 
m
m, you need at least 
m
+
1
m+1 points to determine it uniquely.
In our JSON format each point is stored like:
"2": { "base": "2", "value": "111" }
This means point 
(
x
=
2
,
y
=
111
b
a
s
e
2
=
7
b
a
s
e
10
)
(x=2,y=111 
base2
​	
 =7 
base10
​	
 ).
keys.k is the minimum number of points required to determine the polynomial (k = degree + 1). keys.n is the total number of points provided (n ≥ k). We pick k points, reconstruct the polynomial (or compute P(0)), and then verify the polynomial matches the remaining points.
Why we used Lagrange interpolation (and not other methods)
The core idea: Lagrange interpolation
Lagrange interpolation gives a direct explicit formula for a polynomial that passes through a set of points 
(
x
i
,
y
i
)
(x 
i
​	
 ,y 
i
​	
 ). The polynomial is:
P
(
x
)
=
∑
i
=
0
k
−
1
y
i
⋅
L
i
(
x
)
where
L
i
(
x
)
=
∏
j
≠
i
x
−
x
j
x
i
−
x
j
P(x)= 
i=0
∑
k−1
​	
 y 
i
​	
 ⋅L 
i
​	
 (x)whereL 
i
​	
 (x)= 
j

=i
∏
​	
  
x 
i
​	
 −x 
j
​	
 
x−x 
j
​	
 
​	
 
To get the constant term 
P
(
0
)
P(0), we simply evaluate 
P
(
0
)
P(0):
P
(
0
)
=
∑
i
y
i
⋅
L
i
(
0
)
where
L
i
(
0
)
=
∏
j
≠
i
−
x
j
x
i
−
x
j
P(0)= 
i
∑
​	
 y 
i
​	
 ⋅L 
i
​	
 (0)whereL 
i
​	
 (0)= 
j

=i
∏
​	
  
x 
i
​	
 −x 
j
​	
 
−x 
j
​	
 
​	
 
Why Lagrange for this project
Direct & Conceptually Simple: We need 
P
(
0
)
P(0). Lagrange yields 
P
(
0
)
P(0) directly without computing all coefficients first.
Exact arithmetic friendly: We can compute each term as an exact rational (BigInt numerator and denominator) and keep exact results using gcd reduction—no floating point error.
Minimal code for small k: For typical problem sizes (k fairly small), Lagrange is straightforward to implement and reason about.
Alternatives and why we didn’t pick them (brief)
Solving linear system / Gaussian elimination on Vandermonde matrix:
Would compute coefficients directly, but requires solving a linear system (with rational arithmetic to be exact). It is more code and intermediate matrices can get large. Lagrange computes 
P
(
0
)
P(0) more directly with fewer steps.
Modular interpolation + CRT:
Great for very large inputs or contest problems (fast, avoids giant integers). Requires selecting suitable primes and CRT combination. More complex to implement and reason about; we chose exact BigInt arithmetic for clarity and correctness.
Numeric least-squares / polynomial regression:
For noisy data (approximate fit) this is appropriate. But the problem requires exact polynomial fitting, not approximate fits — so numeric methods are inappropriate here.
Incremental Newton form:
Also valid (Newton’s divided differences). Newton’s method is a good alternative and can be implemented to compute 
P
(
0
)
P(0), but Lagrange is simpler to present when explaining the formula for 
P
(
0
)
P(0).
Key implementation choices (what we did & why)
1. Always use BigInt for values
Some value strings encode very large integers. Regular JS Number loses precision beyond 2^53. Using BigInt ensures exact conversions and arithmetic.
2. Support bases up to 62
Input digits can be 0–9, a–z, A–Z. We map:
'0'..'9' → 0..9
'a'..'z' → 10..35 (this keeps lower-case hex familiar mapping 'a'→10')
'A'..'Z' → 36..61 (extra digits for bases > 36)
This mapping is deterministic and compatible with typical hexadecimal strings (lowercase or uppercase).
3. Exact rational arithmetic with gcd reduction
Lagrange interpolation produces rational numbers when evaluated at 0. We keep a running rational sum sumNum/sumDen and reduce by gcd at every addition to keep numbers smaller and exact.
At the end we check sumNum % sumDen === 0 to make sure the constant term is an integer; otherwise the input is inconsistent or not from a polynomial with integer coefficients.
4. Verification step
We compute the constant term from k selected points (sorted by x, deterministic), then verify the polynomial (via Lagrange evaluation) matches all provided n points.
If verification fails, we reject: either input is noisy/inconsistent or the chosen k points were not the correct subset.
How the code works — high level
Parse JSON:
Read keys.n and keys.k.
For each point key (like "1", "2"), parse base and value and convert value to BigInt.
Select k points:
Sort points by x and pick the first k points (deterministic). (You may choose a different selection policy if needed.)
Compute P(0):
Use Lagrange interpolation evaluated at x=0 with exact rational arithmetic.
Maintain sumNum/sumDen and reduce by gcd every step.
Verify:
Evaluate the polynomial determined from selected points at all given x and confirm the y match. If not, raise an error.
Return:
The constant term as a BigInt (converted to string for display/output).
Example (walkthrough)
Input JSON:
{
  "keys": { "n": 4, "k": 3 },
  "1": { "base": "10", "value": "4" },
  "2": { "base": "2", "value": "111" },
  "3": { "base": "10", "value": "12" },
  "6": { "base": "4", "value": "213" }
}
Decoded points:
(
1
,
4
)
(1,4)
(
2
,
7
)
(2,7) because 111 base 2 = 7
(
3
,
12
)
(3,12)
(
6
,
39
)
(6,39) because 213 base 4 = 24^2 + 14 + 3 = 39
k = 3 → polynomial degree = 2 (quadratic). Use points (1,4), (2,7), (3,12).
Solve for polynomial:
Either solve the 3 equations or use Lagrange. Lagrange evaluation at 0 yields constant term 3:
The polynomial is 
P
(
x
)
=
x
2
+
3
P(x)=x 
2
 +3
Check 
P
(
1
)
=
1
+
3
=
4
P(1)=1+3=4, 
P
(
2
)
=
4
+
3
=
7
P(2)=4+3=7, 
P
(
3
)
=
9
+
3
=
12
P(3)=9+3=12, 
P
(
6
)
=
36
+
3
=
39
P(6)=36+3=39.
So c = 3.
How to run the Node solver (CLI)
Save input JSON (e.g., input.json). From the solver directory:
node solver.js input.json result.txt
# output: result.txt will contain "Constant term c = 3" (or the computed BigInt)
The React UI PolynomialSolverAnalyzer is bundled separately; open your React app and include the component to test in a browser.
Complexity and performance notes
Building each Lagrange term for k points requires O(k) multiplications; evaluating all k terms is O(k^2) arithmetic operations. Each arithmetic operation is on BigInts and may be expensive if numbers are huge.
For small-to-moderate k (e.g., k ≤ 50), the approach is practical. For large k or huge-digit values:
Consider modular approaches: compute results modulo several primes and reconstruct with CRT.
Or use more advanced algorithms (fast multipoint evaluation, Newton interpolation) if performance is critical.
Limitations & failure modes
Noisy/inconsistent data: If the supplied points do not lie exactly on a polynomial with integer coefficients, the solver will either throw (non-integer constant) or report verification failure. This is intentional: the problem expects exact points.
Very large k or huge digits: computation may be slow or memory-heavy.
Base rules: this implementation allows bases up to 62 using the mapping described above. Bases bigger than 62 are not supported here.
Extensions you might want
Try different strategies when verification fails (e.g., search for a subset of k points that fits all other points — expensive).
Provide an option to return all polynomial coefficients (not just the constant). That requires solving for coefficients, which can be done by expanding Lagrange basis polynomials or using Gaussian elimination on a Vandermonde system.
For very large inputs, implement modular interpolation and CRT reconstruction for speed and memory.
Add a mode for floating-point / least-squares fit for noisy data.
Troubleshooting (quick)
Error: Digit 'x' invalid for base b: check the value digits match the claimed base.
Constant term not integer or Verification failed: your points are inconsistent or you picked a wrong subset; check input for typos.
Slow behavior: reduce k, or switch to modular approach for large inputs.
