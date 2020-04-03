"""""""""""""""
Getting Started
"""""""""""""""


LESSON1 Introduction
====================

Welcome to Quantopian! The Getting Started Tutorial will guide you through researching and developing a quantitative trading strategy in Quantopian. It covers many of the basics of Quantopian's API, and is designed for those who are new to the platform. All you need to get started on this tutorial is to have some basic Python programming skills.

What is a Trading Algorithm?
----------------------------

A trading algorithm is a computer program that defines a set of rules for buying and selling assets. Most trading algorithms make decisions based on mathematical or statistical models that are derived from research conducted on historical data.

Where do I start?
-----------------

`Get Notebook <https://www.quantopian.com/clone_notebook?id=5a53d1023b88ce715e5bd502&title=Getting_Started_Tutorial_-_Lesson_1>`_

The first step to writing a trading algorithm is to find an economic or statistical relationship on which we can base our strategy. To do this, we can use Quantopian's Research environment to access and analyze historical datasets available in the platform. Research is a Jupyter Notebook environment that allows us to run Python code in units called "cells." For example, the following code plots the daily closing price for Apple Inc. (AAPL), along with its 20 and 50 day moving averages:

.. code-block:: python

    # Research environment functions
    from quantopian.research import prices, symbols

    # Pandas library: https://pandas.pydata.org/
    import pandas as pd

    # Query historical pricing data for AAPL
    aapl_close = prices(
        assets=symbols('AAPL'),
        start='2013-01-01',
        end='2016-01-01',
    )

    # Compute 20 and 50 day moving averages on
    # AAPL's pricing data
    aapl_sma20 = aapl_close.rolling(20).mean()
    aapl_sma50 = aapl_close.rolling(50).mean()

    # Combine results into a pandas DataFrame and plot
    pd.DataFrame({
        'AAPL': aapl_close,
        'SMA20': aapl_sma20,
        'SMA50': aapl_sma50
    }).plot(
        title='AAPL Close Price / SMA Crossover'
    );

To use the above code copy and paste it into a new notebook in Research, or click the Get Notebook button at the top right corner of this lesson. Once you are in Research, press Shift+Enter to run the cell. The output should look like this:

.. image:: https://www.quantopian.com/assets/tutorials/getting_started/getting_started1_l1_screenshot1-3825f3e80008823f0895a8f194f5d325c8ae1edb6635e3d2566fad41b8538bb3.png

In the next lesson we will use `Research <https://www.quantopian.com/notebooks>`_ to explore Quantopian's datasets. Then, we will define our trading strategy and test whether it can effectively predict returns based on historical data. Finally, we will use our findings to develop and test a trading algorithm in the Interactive Development Environment (IDE).

LESSON 2 Data Exploration
=========================

`Get Notebook <https://www.quantopian.com/clone_notebook?id=5a53d1023b88ce715e5bd503&title=Getting_Started_Tutorial_-_Lesson_2>`_

Lessons 2-4 will be conducted in the Research environment. To get set up in Research, create a new notebook or clone the notebook version of this lesson by clicking Get Notebook below.

Research provides utility functions to query pricing, volume, and returns data for 8000+ US equities, from 2002 up to the most recently completed trading day. These functions take an asset (or list of assets) along with a start and end date, and return a pandas `Series <http://pandas.pydata.org/pandas-docs/version/0.18/generated/pandas.Series.html>`_ (or `DataFrame <http://pandas.pydata.org/pandas-docs/version/0.18/generated/pandas.DataFrame.html>`_) indexed by date.
Let's define the period of time we want to explore and use the returns function to query data for AAPL:

.. code-block:: python

    # Research environment functions
    from quantopian.research import returns, symbols

    # Select a time range to inspect
    period_start = '2014-01-01'
    period_end = '2014-12-31'

    # Query returns data for AAPL
    # over the selected time range
    aapl_returns = returns(
        assets=symbols('AAPL'),
        start=period_start,
        end=period_end,
    )

    # Display first 10 rows
    aapl_returns.head(10)

.. image:: https://www.quantopian.com/assets/tutorials/getting_started/getting_started1_l2_screenshot1-ecd31677096c10753ee59b00075b4f5970d702a839d43646ca4a9c75ce598053.png

Alternative Data
----------------

In addition to pricing and volume data, Quantopian integrates a number of other datasets that include corporate fundamentals, stock sentiment analysis, and consensus estimates, to name a few. You can find the complete list of datasets in Quantopian's `Data Reference <https://www.quantopian.com/docs/data-reference/overview>`_.
Our goal in this tutorial will be to build an algorithm that selects and trades assets based on sentiment data, so let's take a look at PsychSignal's `StockTwits Trader Mood dataset <https://www.quantopian.com/docs/data-reference/psychsignal>`_. PsychSignal's dataset assigns bull and bear scores to stocks each day based on the aggregate sentiment from messages posted on Stocktwits, a financial communications platform.
We can start by inspecting the message volume and sentiment score (bull minus bear) columns from the stocktwits dataset. We will query the data using Quantopian's Pipeline API, which is a powerful tool you will use over and over again to access and analyze data in Research. You will learn a lot more about the Pipeline API in the next lesson and a `later tutorial <https://www.quantopian.com/tutorials/pipeline>`_. For now all you need to know is that the following code uses a data pipeline to query stocktwits and returns data, and plots the results for AAPL:

.. code-block:: python

    # Pipeline imports
    from quantopian.research import run_pipeline
    from quantopian.pipeline import Pipeline
    from quantopian.pipeline.factors import Returns
    from quantopian.pipeline.data.psychsignal import stocktwits

    # Pipeline definition
    def make_pipeline():

        returns = Returns(window_length=2)
        sentiment = stocktwits.bull_minus_bear.latest
        msg_volume = stocktwits.total_scanned_messages.latest

        return Pipeline(
            columns={
                'daily_returns': returns,
                'sentiment': sentiment,
                'msg_volume': msg_volume,
            },
        )

    # Pipeline execution
    data_output = run_pipeline(
        make_pipeline(),
        start_date=period_start,
        end_date=period_end
    )

    # Filter results for AAPL
    aapl_output = data_output.xs(
        symbols('AAPL'),
        level=1
    )

    # Plot results for AAPL
    aapl_output.plot(subplots=True);

.. image:: https://www.quantopian.com/assets/tutorials/getting_started/getting_started1_l2_screenshot2-fda42689e3ebaa6061d9a70fb1395a6dd90d2b5cb6fd49ce41f2978416843d2a.png

When exploring a dataset, try to look for patterns that might serve as the basis for a trading strategy. For example, the above plot shows some matching spikes between daily returns and stocktwits message volume, and in some cases the direction of the spikes in returns match the direction of AAPL's sentiment score. This looks interesting enough that we should conduct more rigorous statistical tests to confirm our hypotheses.
In the next lesson we will cover the Pipeline API in more depth.