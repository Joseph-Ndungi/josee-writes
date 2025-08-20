---
title: "Portfolio Optimization"
date: 2024-10-03
categories: [Analytics]
tags: [Portfolio Optimization, analytics]
canonical_url: "https://www.innova.co.ke/portfolio-optimization/"
---

# Portfolio Optimization

## Introduction

Portfolio optimization is the method of selecting the best portfolio, which gives back the most profitable rate of return for each unit of risk taken by the investors. ​

A portfolio is defined as the pool of investment options of an investor. ​

The best portfolio for an investor depends upon various options like risk appetite, expected rate of return, other cost minimization.​

An ideal portfolio would be the one that has the highest Sharpe ratio.​

Optimization of a portfolio is based upon the modern portfolio theory (MPT), which affirms that investors want the highest rate of return with the lowest risk.

To achieve the principle of MPT, the assets need to be diversified or have the least correlation to each other. ​

This is to say that the impact of the low performance of one asset would not result in the crash of the entire portfolio. 

## How to Optimize a Portfolio​

Step 1: Selection of Asset classes: Portfolio managers first choose asset classes for investing the client’s or investor’s funds. ​

This selection depends upon the factors like the expected rate of return needed by the investor and the risk appetite of the investor.  

Step 2: Selection of Assets within the class: The portfolio manager will choose how much bonds or equity will be included in the portfolio. ​

What kinds of shares will form part of the equities in the portfolio, and how much money will be invested in real estate and fixed-interest securities? This choice is also based on the risk-return relationship chosen by the investor.

![][image_ref_s37dsqe6]

The X-axis (representing standard deviation) corresponds to the portfolio's return regarding the level of risk. When we combine this portfolio with a risk-free asset, the point on this graph where the Sharpe ratio is maximized signifies the optimal portfolio. It marks the juncture at which the capital allocation line is tangential to the efficient frontier. The rationale behind this is that at that point, the Sharpe ratio (which quantifies the increment in expected return for each additional unit of risk incurred) is at its zenith, however, this does not imply that all portfolios will achieve similar outcomes.

## Portfolio Optimization Models​

* Mean-variance model: Through this model, the portfolio manager is able to determine the optimal portfolio, which is the point at which the risk-return trade-off is highest in the graphical representation in the efficient frontier. It is based upon several assumptions relating to the market and investors.​

* Mean semi-variance model: Mean semi-variance model helps in measuring the potential downside risk of an optimized portfolio. The average standard deviation of values that falls below the mean is termed semi-variance.​

* Conditional Value at Risk model (CVAR): CVAR model helps in measuring the tail risk associated with the investment portfolio. The technique is used for effective risk management in the portfolio optimization process.​

* Mean absolute deviation model: Mean absolute deviation model is used while selecting the large-scale optimized portfolio.

## Comparison of the models and the best use cases​

* Mean-Variance (MV): Best for investors looking for a simple, well-established method to balance risk and return.​

* Semi-Mean Variance (SMV): Best for risk-averse investors who are particularly concerned with downside risk. ​

* Value at Risk (VaR): Best for regulatory and reporting purposes, providing a clear risk metric. ​

* Conditional Value at Risk (CVaR): Best for investors and risk managers who need to understand and manage extreme risks.

## Advantages of portfolio optimization​

* [ ] Portfolio optimization helps in the maximization of the return on investment. This is done through the efficient frontier graph, which depicts the point where the risk-return trade-off for the portfolio is highest; that point gives back the optimal portfolio.​

* [ ] Portfolio optimization helps in the diversification of the portfolio. The portfolio manager chooses the diversified portfolio so that one under performing asset does not impact the complete portfolio.​

* [ ] For selecting the optimal portfolio, portfolio managers do a lot of market research which helps them in identifying market opportunities before other people, which ultimately benefits their clients or investors
.

## Disadvantages​

* [ ] Some assumptions taken by portfolio managers while choosing an optimal portfolio, like a friction-less market, are untrue in reality; there are frictions like transaction cost and other constraints that complicate the process of portfolio optimization.​

* [ ] The assumptions can throw back incorrect results, which could result in the loss of money.​

* [ ] There is always a risk of over-diversification of the assets, which would result in a marginal loss of expected return more than the marginal benefit of the reduced risk. This would ultimately result in erosion of the investor’s benefit of the expected rate of return on the investment.

## Portfolio Optimization in Discrete time​

Portfolio analysis involves assessing the performance and risk of a collection of financial assets, such as stocks, bonds, or other investments.​

In a discrete time setting, the analysis is conducted at specific intervals (e.g., daily, monthly, quarterly), rather than continuously.​

Discrete time models have been extensively used in finance due to their simplicity and ease of implementation.​

They provide a practical approach for portfolio management, especially when dealing with data that is naturally recorded at discrete intervals.

**Key Concepts:**

Returns: Measurement of the gain or loss of an asset over a specific time period.​

Risk: Quantification of uncertainty in the returns, often measured by metrics such as variance or standard deviation.​

Diversification: Strategy of combining various assets to reduce overall portfolio risk.​

Re-balancing Strategies: Determining the optimal times and methods for adjusting the portfolio to maintain a desired level of risk and return.​

Performance Attribution: Breaking down the portfolio's returns to understand the contribution of different assets and investment strategies.


## Importance of Discrete Time Models in Financial Decision-Making​

* Financial markets operate in discrete time, with data reported at regular intervals (e.g., end-of-day prices).​
* Decision-makers, such as portfolio managers and investors, often make investment decisions based on discrete data.​
* Discrete time models are computationally less intensive compared to continuous time models.​
* They are easier to implement using standard computational tools and techniques​
* Most financial data, such as historical prices, dividends, and interest rates, are available in discrete time.​
* Regulatory and reporting requirements often mandate discrete time analysis (e.g., quarterly earnings reports).​
* Discrete time analysis helps in the periodic assessment of portfolio risk, enabling timely adjustments to the investment strategy.​
* Tools like Value at Risk (VaR) and Conditional Value at Risk (CVaR) can be effectively applied in a discrete time setting.​
* Discrete time models provide a clear framework for portfolio optimization, helping investors allocate resources effectively.​
* They allow for the incorporation of various constraints and investor preferences in the optimization process.


## Discrete-time portfolio analysis tools and techniques​

These tools are essential for managing and assessing investments over time, as they allow investors to make decisions based on periodic reviews of the portfolio’s performance.​

1. Monte Carlo simulation ​

* Monte Carlo simulation is a statistical technique used to model and analyze the behavior of systems that involve uncertainty and random processes. ​
* It is used to simulate the performance of a portfolio over time by generating a large number of potential future scenarios based on random sampling.
​

2. Numerical Optimization​

* Numerical optimization involves the use of mathematical algorithms to find the best possible solution to a problem within a given set of constraints. ​
* Numerical optimization techniques are used to determine the optimal allocation of assets that maximize returns or minimize risk.

## Common methods in numerical optimization​

* Mean-Variance Optimization: Finding the portfolio that offers the highest expected * return for a given level of risk, or equivalently, the lowest risk for a given level of expected return.​
* Quadratic Programming: Solving optimization problems where the objective function is quadratic and the constraints are linear, which is a common setup in portfolio optimization.​
* Non-linear Optimization: Handling more complex portfolio optimization problems where the relationships between variables are non-linear, often used for incorporating realistic market conditions and constraints.


[image_ref_s37dsqe6]: data:image/png;base64,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
