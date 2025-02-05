---
layout: post
title: Tidymodels hyperparameter tuning with targets dynamic branching
subtitle: Dynamic branching with tune_grid()
thumbnail-img:  /assets/img/tidy_and_targets.png    
tags: [data, R, tricks]
mathjax: true
---

## Machine learning model parameters

At it’s core machine learning is a collection of statistical and
mathematical techniques to analyze and draw inferences from patterns in
data. The way machine learning models ‘learn’ is by adjusting sets of
parameters in order to minimize error and maximize predictive
performance. For example, in linear regression, the parameters are the
regression coefficients $$\mathbf{\beta}$$ and the intercept. For neural
networks, the parameters are the weights and biases of each neron. In
tree based models, the parameters include the collection of tree
topologies and their associated leaf weights.

## Parameters vs hyper-parameters

While model parameters are learned from the data during training,
hyper-parameters control *how* models learn from the data. For example,
[xgboost](https://xgboost.readthedocs.io/en/stable/tutorials/model.html)
hyperparameters include $$\mathbf{\gamma}$$ a regularization parameter
that controls the cost of tree complexity and $$\mathbf{max_depth}$$
which limits how deep each tree can grow.

Hyper-parameters must be specified prior to model fitting, but how to
chose the best ones? This is where cross-validation comes in. By
splitting the data into training, validation, and testing datasets it’s
possible to tune the hyper-parameters on a subset of the data and then
fit and evaluate the final model performance against a hold-out data
set.

## Parallel hyper-parameter tuning

Hyper-parameter tuning is an important step in machine learning
pipelines. It generally involves fitting a lot of models across a grid
of different hyper-parameters in order to find the optimal set.
Unfortunately fitting so many models takes a lot computational resources
and time. Fortunately the problem is what is commonly known as
[embarrassingly
parallel](https://en.wikipedia.org/wiki/Embarrassingly_parallel). That
means each of the models can be fit independently and in parallel.

## Tidymodels tune_grid()

In the [tidymodels](https://www.tidymodels.org/) package in R,
hyper-parameter tuning is handled by the **`tune_grid()`** function. The
**`tune_grid()`** function can handle parallelism using the
[future](https://future.futureverse.org/) package in R. However there
are some limitations, for example there isn’t an easy way to monitor
progress. Another problem is that if one of the models stochastically
fails it can disrupt the entire tuning process. Lastly it can be useful
to control whether parallelization occurs over the cross-validation
folds, over the list of hyper-parameter sets, or a cross of both.

## Targets dynamic branching

Fitting a lot of models involves orchestrating many different
processes - something that pipeline management tools like
[targets](https://books.ropensci.org/targets/) are specifically designed
to do. In targets, fitting a lot of models at once can be accomplished
using [dynamic
branching](https://books.ropensci.org/targets/dynamic.html). Dynamic
branching sets up a subprocess for each model. Targets can track the
progress of branches as they complete using **`tar_watch()`** and
**`tar_poll()`** and any failed branches can be re-run after tuning
completes by setting `error = "null"` within the dynamically branched
target. The `pattern` argument in the target can also be used to specify
how branching occurs - mapping over the cross validation folds, the
hyper-parameters, or using `cross()` to fit a model to every
combination.

## targets and tidymodels

Below is an example targets based tidymodels pipeline. It uses two
custom functions, **`tune_grid_branch()`** and **`select_best_params`**,
within a dynamically branched target to tune the hyper-parameters of a
Bayesian Additive Regression Trees (BART) model. The key target is
`bart_tuned` which is set to branch over every combination of
cross-validation fold and hyper-parameter set combination via
`pattern = cross(training_data_folds, bart_gridsearch)\`.

      tar_target(analysis_recipe, recipe(paste(response_variable, "~ .") |>
    as.formula(), data = analysis_data_train) |>
                   step_naomit() |>
                   step_string2factor(all_string()) |>
                   step_novel(all_nominal(), -all_outcomes()) |>
                   step_dummy(all_nominal(), -all_outcomes()) |>
                   step_zv(all_predictors())),
                   
    # Set up the BART model
      tar_target(bart_model, 
                 parsnip::bart(trees = tune(),
                               prior_terminal_node_coef = tune(),
                               prior_terminal_node_expo = tune()) |> 
                   set_engine("dbarts") |>
                   set_mode("classification")),
      
      # Set up the BART model workflow
      tar_target(bart_workflow, workflow() |> 
                   add_recipe(analysis_recipe) |> 
                   add_model(bart_model)),
      
      # Set up the hyper-parameter grid search.
      # Automatically extract the parameters to tune across.
      tar_target(bart_gridsearch, bart_workflow |> 
                   extract_parameter_set_dials() |>
                   dials::grid_latin_hypercube(size = 10)),
      
      # Tune the model
      tar_target(bart_tuned, tune_grid_branch(workflow = bart_workflow,
                                               gridsearch_params = bart_gridsearch,
                                               training_data_folds = training_data_folds),
                 pattern = cross(training_data_folds, bart_gridsearch))
                 
      # Extract the best set of hyper-parameters
      tar_target(bart_best_params, select_best_params(bart_tuned, metric = "roc_auc")),
      

## tune_grid_branch()

Below is my implementation of a dynamic branch friendly
**`tune_grid()`** function. In addition to the benefits described above
it also tracks both fit time and the amount of memory required for each
model fit. With targets it’s also possible to just fit the first few
fold and hyper-parameter combinations in order to profile your model
tuning workflow. This can be accomplished by setting
`pattern = head(cross(training_data_folds, bart_gridsearch), 5)` which
will fit the first 5 models only. You can then get a sense of how long
the fitting will take and how much resources each branch will require.
The best part is that when you’re ready to do the full run you can
remove the `head()` part of the command and targets *won’t need to
re-run those first 5 models*. I’m certain there’s room to improve this
but at least to me it seems like a very powerful combination.

    #' Grid search for a tidymodels workflow using targets dynamic branching
    #'
    #' This function performs grid search tuning for a machine learning workflow 
    #' using cross-validation. It iterates over provided folds and grid search 
    #' parameters and computes specified evaluation metrics (e.g., AUC, F1 score) 
    #' and profiles memory usage and timing for each model fit.
    #' 
    #' @author Nathan Layman
    #'
    #' @param workflow # A tidymodels workflow with recipe and model already attached
    #' @param gridsearch_params # A tibble where each row is a set of hyperparameters
    #' @param training_data_folds # A tibble where each row is a training data fold
    #' @param metrics # A set of metrics produced by `yardstick::metric_set`
    #'
    #' @return Returns a tibble with fit performance metrics, fit time, and the ram used while fitting
    #'
    #' @examples
    #' \dontrun{
    #' performance <- tune_grid_branch(workflow, gridsearch_params, training_data_folds, verbose = TRUE)
    #' }
    #'
    #' @export
    tune_grid_branch <- function(workflow, 
                                 gridsearch_params, 
                                 training_data_folds, 
                                 metrics = metric_set(pr_auc,           # Precision-Recall AUC
                                                      roc_auc,          # ROC AUC
                                                      accuracy,         # Accuracy
                                                      f_meas,           # F1 Score
                                                      recall,           # Recall
                                                      precision),
                                 verbose = F) {
      
      # Get the performance and profiling metrics of every combination of 
      # data fold and hyper-parameter combination passed in to the function
      performance <- map_dfr(1:nrow(training_data_folds), function(i) {
        map_dfr(1:nrow(gridsearch_params), function(j) {
          
          rsamp <- rsample::manual_rset(training_data_folds[i,]$splits, training_data_folds[i,]$id)
          params <- gridsearch_params[j,]
          if(verbose) print(rsamp |> bind_cols(params) |> select(-splits))
          
          # Grab start time
          start_time <- Sys.time()
          
          # Fit the model against the training data and profile memory usage
          # Using tune_grid here but we could make this simpler and just
          # fit and evaluate the model manually
          mem_usage_bytes <- profmem::profmem({
            fold_param <- tune::tune_grid(workflow,
                                          resamples = rsamp,
                                          grid = params,
                                          metrics = metrics)
          })
          
          # Report fit performance metrics
          fold_param |> select(-splits) |>
            mutate(id = rsamp$id,
                   branch = targets::tar_name(),
                   mem_usage_bytes = sum(mem_usage_bytes$bytes, na.rm=T),
                   fit_time = start_time - Sys.time())
        })
      })
      
      # Clean up environment in case targets tries to store extra stuff
      rm(list=setdiff(ls(), "performance"))
      
      # Return performance
      performance
    }

## select_best_params()

And here is how you select the best set of parameters from the tibble
returned by `tune_grid_branch`. This should seamlessly fit back into a
tidymodels pipeline for fitting the final model, extracting performance
metrics against the hold-out test dataset, and performing variable
importance (e.g. DALEX).

    #' Select Best Parameters from Tuned Model Results
    #'
    #' This function extracts the best parameters from a tuned model's metrics based on a specified evaluation metric. 
    #' It calculates the average of the specified metric across tuning folds and selects the parameters with the 
    #' minimum value of the specified metric (e.g., "roc_auc"). Unnecessary columns such as splits, IDs, and memory usage are removed.
    #'
    #' @author Nathan Layman
    #'
    #' @param tuned A tibble containing the results of the tuning process, including the model metrics.
    #' @param metric A character string specifying the evaluation metric to be used for selecting the best parameters. 
    #' The default is `"roc_auc"`.
    #'
    #' @return A tibble containing the best parameters, excluding unnecessary columns such as `.estimate`, `mem_usage`, and any matching branches.
    #' 
    #' @details The function first unnests the `.metrics` column of the `tuned` tibble, filters by the selected metric, 
    #' and calculates the mean of the evaluation metric for each set of parameters. It then selects the parameters 
    #' that minimize the metric, without ties.
    #'
    #' @examples
    #' \dontrun{
    #' # Example usage:
    #' best_params <- select_best_params(tuned_model_results, metric = "roc_auc")
    #' }
    #'
    #' @export
    select_best_params <- function(tuned, metric = "roc_auc") {
      
      best_params <- tuned |> 
        unnest(.metrics) |> 
        filter(.metric == metric) |> 
        select(-splits, -id, -starts_with("."), .estimate) |>
        group_by(across(-.estimate)) |>
        summarize(.estimate = mean(.estimate), .groups = "drop") |>
        slice_min(.estimate, with_ties = F) |>
        select(-.estimate, -starts_with("mem_usage"), -matches("branch"))
      
      return(best_params)
    }
