---
title: "Machine Learning using SQL?"
date: 2020-01-26T12:00:00-07:00
categories: ["Data Science","Analytics"]
draft: false
---

{{< featured-image 
    src="https://miro.medium.com/v2/resize:fit:1224/format:webp/0*cC_4db2J_VoWbNoy"
    sizes="(max-width: 2400px) 100vw, 2400px" 
    alt="Machine Learning (Source: https://www.istockphoto.com/in/photos/machine-learning)" 
>}}

{{< lead >}}
"The journey of a thousand miles begins with one step." — Lao Tzu
{{< /lead >}}

{{< dropcap >}}
Yes, you read that one right! One of the most fundamental machine learning algorithms out there is Linear Regression. In simple words, it is a technique to describe a relationship between a response(a.k.a dependent) variable and one or more explanatory (a.k.a independent) variable(s). After doing some reading into the “math” behind these algorithms, I realized that this can be easily achieved in SQL.
{{< /dropcap >}}


I don’t intend to turn this post into another “Machine Learning 101”. There are plenty of such articles out there that explain what Linear Regression is in a much better way, including some nitty-gritty details like it’s back-end optimization algorithm, i.e. Gradient Descent. I will thus try to keep this article as less “ML”ly as possible. However, I’d recommend this [9 byte-sized (typo intended!) article series](https://www.internalpointers.com/post/introduction-machine-learning) that explains all this and some more in a really easy language. Also, for most of this post, I will be referring to the formulas and notations used in the hyperlinked article.

---

Alright! At this point, I hope that you already know about the concepts of Linear Regression and how Gradient Descent works. And thus you’ll also know that the relationship between response and explanatory variable(s) is explained by the following equation :

{{< responsive-image src="https://miro.medium.com/v2/resize:fit:594/format:webp/1*SY9gSMDpHJedmu7q3mWSbg.png" alt="Hypothesis function for linear regression" max-width="100%" caption="Hypothesis function for linear regression" >}}

The goal of Linear Regression is to find the optimal value of θ (theta) that best describes the relationship between two variables and Gradient Descent is the way to do that. The equation given below summarizes how Gradient Descent arrives at the optimal value of θ(s).

{{< responsive-image src="https://miro.medium.com/v2/resize:fit:968/format:webp/0*Pl5ijyxkj-zzPn0z.png" alt="Gradient Descent" max-width="100%" caption="Gradient Descent Equation" >}}

---

### Getting down to business:

{{< responsive-image src="https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExNjNiYm9xamZtM3N4a3JlamhsYmZyaWx6YzBrdjRmcGFpc3JzNzc5OSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/6FwsjocwLc4us/giphy.webp" alt="Giphy" max-width="100%" >}}

Let’s list down the activities that are needed to be performed to arrive at optimal values of θ:

1. Start with random values of θ and calculate the value of hypothesis function (Hθ)
2. Fill in the values of θ and Hθ in the convergence equation to get new values of θ
3. Keep repeating Step 2 until the values of θ don’t change anymore
4. These values of θ correspond to the minimum cost (or error) for the Hypothesis function
5. Fill in the final values of θ in the hypothesis function to get Predicted values of the response variable

##### Step 1: Declaring & Initializing variables

We need 6 variables, each for a different purpose:

1. theta0, theta1 to store the current value of θ0, θ1
2. theta0_t, theta1_t to store temporary values of θ0 & θ1 before updating the original ones
3. var (short for variability) to check if the updated value of θ is approaching “near” the current value or not
4. alpha to store learning rate (read this answer at Quora to understand all about learning rate)

```SQL
DECLARE @theta0 int;
DECLARE @theta1 int;
DECLARE @theta0_t int;
DECLARE @theta1_t int;
DECLARE @var DECIMAL(4,3);
DECLARE @alpha DECIMAL(4,2);

--Initial values
SET @theta0 = 0;
SET @theta1 = 1;
SET @theta0_t = 1;
SET @theta1_t = 0;
SET @alpha = 0.1;
SET @var = 0.01;
```

##### Step 2: Calculating values of Hθ and updated values of θ

```SQL
--Calculating theta0
SELECT
 @theta0_t = @theta0 - (SUM(Outp)/(SELECT COUNT(*) FROM base))*@alpha
FROM
 (
 SELECT
  (@theta1*X + @theta0) - Y as Outp
 FROM base
 );
--Calculating theta1
SELECT
 @theta1_t = @theta1 - (SUM(Outp)/(SELECT COUNT(*) FROM base))*@alpha
FROM
 (
 SELECT
  ((@theta1*X + @theta0) - Y)*X as Outp
 FROM base
 );
```

##### Step 3: Comparing if the updated values of θ are close to original θ or not

```SQL
--Comparing thetas
IF (@theta0_t BETWEEN @theta0-@var AND @theta0+@var) AND (@theta1_t BETWEEN @theta1-@var AND @theta1+@var)
```

If the above condition is true, then we stop the process and finalize the values of θ. Otherwise, we keep repeating steps 2 & 3. Thus steps 2 & 3 need to be put inside a loop that runs as long as the updated and current values of θ are different.

```SQL
WHILE (@theta0_t NOT BETWEEN @theta0-@var AND @theta0+@var) AND (@theta1_t NOT BETWEEN @theta1-@var AND @theta1+@var)
BEGIN
	--Calculating theta0
	SELECT
		@theta0_t = @theta0 - (SUM(Outp)/(SELECT COUNT(*) FROM base))*@alpha
	FROM
		(
		SELECT
			(@theta1*X + @theta0) - Y as Outp
		FROM base
		);

	--Calculating theta1
	SELECT
		@theta1_t = @theta1 - (SUM(Outp)/(SELECT COUNT(*) FROM base))*@alpha
	FROM
		(
		SELECT
			((@theta1*X + @theta0) - Y)*X as Outp
		FROM base
		);
	
	--Comparing thetas
	IF (@theta0_t BETWEEN @theta0-@var AND @theta0+@var) AND (@theta1_t BETWEEN @theta1-@var AND @theta1+@var)
	BEGIN
		SELECT @theta0 = @theta0_t;
		SELECT @theta1 = @theta1_t;
		BREAK;
	END
	ELSE
	BEGIN
		SELECT @theta0 = @theta0_t;
		SELECT @theta1 = @theta1_t;
	END
END
```

The above loop will arrive at optimal values for θ. This is Gradient Descent in all it’s glory!

{{< responsive-image src="https://media0.giphy.com/media/6v5ShFHPRF0pa/200w.webp?cid=dda24d50q3itazzy8noerq4kd6mn8xsrklpaingkexf0036a&ep=v1_gifs_related&rid=200w.webp" alt="Giphy" max-width="100%" >}}

##### Step 4: Fill in the final values of θ in the hypothesis function to calculate predictions for the response variable

```SQL
SELECT X,Y,@theta0+@theta1*X AS H_theta
FROM base
```

And that’s it! We’ve built a machine learning algorithm in SQL with just a few lines of code!

Refer to this [GitHub repository](https://github.com/kohlisaab/ML_in_SQL/tree/master) for the entire code.

---

### Practical applications & final thoughts
Despite the onset of technological advancements in the field of Data Science, more often than not, every Data Scientist ends up working with legacy systems. In such cases, if the size of the data is huge, it becomes impractical to fetch it out of a legacy system (like SQL Server) into another environment for data science purposes.

Although I initially began this project as a weekend DIY, I feel this has bigger implications. This can be polished and packaged much better to improve its usability. Things like splitting of data into test & train, turning this into multi-variate linear regression will make this project much more practical. I would also love to hear thoughts of all of you on what can be improved.

I am now planning to write more articles (when I manage to get enough coffee in me) in this series, where I will attempt to implement other commonly used ML algorithms purely in SQL.

Happy reading!