---
title: Eliminates the Left Recursion
---

#### Direct

$$
\begin{aligned}
  A \to A \alpha \;/\; \gamma &\iff A \to \gamma A' \land A' \to \alpha A' \;/\; \epsilon \\
  A \to A \alpha \;/\; A \beta \;/\; \gamma &\iff A \to \gamma A' \land A' \to \alpha A' \;/\; \beta A' \;/\; \epsilon
\end{aligned}
$$


#### Indirect

$$
\begin{aligned}
  &\phantom{\iff} S \to Q c \;/\; c \land Q \to R b \;/\; b \land R \to S a \;/\; a \\
  &\iff S \to Q c \;/\; c \land Q \to (S a \;/\; a) b \;/\; b \\
  &\iff S \to ((S a \;/\; a) b \;/\; b) c \;/\; c \\
  &\iff S \to (S a b \;/\; a b \;/\; b) c \;/\; c \\
  &\iff S \to S a b c \;/\; a b c \;/\; b c \;/\; c \\
  &\iff S \to a b c S' \;/\; b c S' \;/\; c S' \land S' \to a b c S' \;/\; \epsilon \\
  &\phantom{\iff Q \to S a b \;/\; a b \;/\; b} \\
  &\phantom{\iff R \to S a \;/\; a}
\end{aligned}
$$

Notice the $$ Q $$ and $$ R $$ are not accessible after eliminating the left recursion of $$ S $$, thus we'd better make them not the token we want, though it may be difficult..
