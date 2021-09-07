# Data Transformation

## KNMI data

For this first assignment we will be using KNMI weather data, containing
average temperatures from *1901* through *2016*. The question we are trying
to answer with this data is:

* Can we find any visually compelling evidence for climate change in the form
of increasing average temperatures in the Netherlands over the last century?

Get started by downloading the data [here](tas_NLD.zip) and creating a Jupyter
Notebook named `transformation_KNMI.ipynb`.

### Loading the data

Load each of the files using the *pandas* [read_csv()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html)
function. Merge each of the resulting data frames together to a single data
frame using a [concat()](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html#concatenating-objects)
operation.

Print or output the resulting data frame and consider whether it looks correct. Now, try
to access each of the columns of the frame individually. You should be running
into some unexpected errors, which are caused by the formatting of the data.
Open one of the data files in a text editor and see if you can spot what
exactly is going wrong. The `read_csv` function has several optional arguments
you can add to change how files are processed. Try to find the appropriate
optional argument to change and update your code to load each of the data files
correctly.

### Plotting the data and the trend

Let's start by making a basic plot of the loaded data using
[pyplot](https://matplotlib.org/3.1.0/api/_as_gen/matplotlib.pyplot.plot.html).
The horizontal axis should be the years and the vertical axis the temperature.

It is hard to tell if there is any kind of trend in the data, due to the large
variations between the seasons each year, so we'll try to fit a straight line
through the data to indicate that trend. For this we'll use the
*scikit-learn* package, which contains many useful machine learning models.
Try to import the module with `import sklearn`. If the package is not installed yet,
follow the instructions [here](https://scikit-learn.org/stable/install.html).

[Linear Regression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html)
is one of the most simple machine learning models in sklearn, so it should serve as a good
introduction. Start by making a new instance of the model using the following
code.

    from sklearn.linear_model import LinearRegression

    model = LinearRegression()

Now you can call functions on this model like `model.fit(X, y)` and
`model.predict(X)` Use this [fit()](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html#sklearn.linear_model.LinearRegression.fit)
function to fit a line through your data, where `X` should be the input data,
so in this case the *years*, and `y` the target data, which should be
the *temperature*.

Trying to use the `fit` function will probably result in the following
error: `ValueError: Expected 2D array, got 1D array instead`. This is because
the `fit` function is designed to handle multiple input features for the model,
which means that it expects the data in a specific 2D format. It will be a lot
easier to change the shape of the data if we convert the years *series* to a
*numpy array*, using the [to_numpy()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.to_numpy.html#pandas.Series.to_numpy)
function and then [reshape()](https://docs.scipy.org/doc/numpy/reference/generated/numpy.reshape.html)
the resulting array to the shape the error message suggests.

Once the input data is in the correct shape, calling the `fit` function will
change the `model` to fit the data we have given it. In order to see the actual
fitted line, we'll need to use the [predict()](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html#sklearn.linear_model.LinearRegression.predict)
function. As we want to show the fit for the same years for which we fitted,
it is possible to just reuse the reshaped array containing the years as the `X` input for
the `predict()` function. The function will return the fitted results for each of
those years.

Add an extra column to your data frame containing the results of fitting the
linear model to the temperatures. Modify your plotting code to plot this fitted
linear line in addition to the original data and finally label both lines.

### Selecting months

There seems to be a slight upward trend in the temperature, but it is still
difficult to see with all the seasonal fluctuations. So, next we'll just plot the
averages for one specific month and try to reduce the seasonal variations in
the plot that way. Use [Boolean indexing](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#boolean-indexing)
to filter out a specific month and plot the temperatures for *June*, *July* and
*August*.

All 3 plots should contain both the temperature data and a fitted line. Modify your
code from the previous step to consist mainly of functions, which you can then
reuse to create these 3 plots. Try to write the functions to be general enough such
that they can be reused, allowing you to avoid a lot of copy pasting of code
for this assignment.

Adding the column containing the results of the fitted line should be done
separately for each of the 3 months, as they will each have a slightly
different trend. Depending on how exactly you solved this part of the
assignment, you might get a `SettingWithCopyWarning`. [This](https://www.dataquest.io/blog/settingwithcopywarning/)
blog post gives a nice general introduction on why you get this warning, then
a lot of detail on how it is caused (which you may skip), and finally a couple
of tips on how to avoid or turn off this warning.

### Averages over larger time periods

With these month selections, the trend starts to be a bit clearer, but there
are also still some pretty big fluctuations in temperature between each year.
So, next we'll compute the average for the whole year, combining each of the
months, and try to smooth out the fluctuations that way. Using [groupby()](https://pandas.pydata.org/pandas-docs/stable/user_guide/groupby.html#splitting-an-object-into-groups)
we can split the data frame according to a property and then
recombine the results with an [aggregation function](https://pandas.pydata.org/pandas-docs/stable/user_guide/groupby.html#aggregation).
Plot the year averages and linear fitted line for the combined data using your
functions from before.

The trend again appears to be a bit clearer, but it could still use some
smoothing. As a last attempt to improve the plot, we'll try and compute the
averages over each decade and plot those. The [groupby()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.groupby.html)
function has quite a few different ways you can group elements. Try to find a
way to use this function to group the data by decades and average them. Plot
the resulting averages and the linear fitted line for those averages.

## Submitting

After finishing, submit your `transformation_KNMI.ipynb` file below.
