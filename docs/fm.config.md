Factorization Machine Model Config:

```json
# filesystem scheme URI
# local filesystem : "local", "file:///", (in linux and os system, you can use both,  but windows local filesystem only support "local")
# hdfs filesystem : "hdfs://host"
# other filesystem URI
fs_scheme : "file:///",

# if you want to see more detailed logs, set verbose to true
verbose : false,

data {
    # train data
    train {
        # train data path
        # local filesystem : supports file, recursive directories
        # hdfs filesystem : depends on spark or hadoop cluster, spark supports regex paths
        data_path : "???", 
  
        # max tolerable error format count in train data
        max_error_tol : 0
    },

    # test/validation data
    test {
        # test data path(if you have no test data, you don't have to supply it)
        # local filesystem : supports local file and recursive directories
        # hdfs filesystem : depends on spark or hadoop cluster, spark supports more complicated paths(more that one path, regex paths)
        data_path : "",
      
        # max tolerable error format count in train data
        max_error_tol : 0
    },

    # delimiters, see data_format.md for more details
    # train/test data format:[]()
    #   regression : weight###label###f1name:f1value,f2name:f2value,...
    #   binary classification : weight###label(0 or 1)###f1name:f1value,f2name:f2value,...
    #   binary cross_entropy : weight###label(0 ~ 1, positive)###f1name:f1value,f2name:f2value,...
    delim {
        # separate sample weight, labels, features
        x_delim : "###",
      
        # if you have more than one label, separated by y_delim
        y_delim : ",",
      
        # separates features(a feature includes feature_name and feature_value)
        features_delim : ",",
      
        # separate feature_name and feature_value
        feature_name_val_delim : ":"
    },

    # if your task is of classification(including multi-class classification), 
    # you can downsample/upsample some special classes in special probability/weight
    # format : y_sampling : ["class1@prob", "class2@prob", ...]
    # downsampling: binary classification, if you want to reserve negative samples in 0.1 prob, set y_sampling to ["0:0.1"]
    # upsampling: multi-class classification, if you want to increase 6'th class sample weight by 10X and 8'th class sample weight by 5X, set y_sampling to ["6@10","8@5"] 
    # empty means no sampling
    y_sampling : [],

    # whether your train/test data is assigned. See "Train/Test Data Splitting Manner" in running_guide.md for more details on train/test data assignment method
    assigned : false,
   # if your train/test data is not assigned, we provide the following two ways for slaves to read files: 
    # lines_avg : different slaves read different lines of same file alternative. If you have a few train/test files and more than one slave, we recommend this manner
    # files_avg : different slaves read different files, if your files outnumber slaves, and the number of samples in each file is similar, we recommend this manner
    unassigned_mode : "lines_avg" // "files_avg"
},

feature {
    # feature hash, implements "Feature Hashing for Large Scale Multitask Learning"
    # if your feature dim very large, training will be very slowly, you can use feature hash to reduce dim at a fraction of cost. 
    feature_hash {
        # switch
        need_feature_hash : false,
  
        # feature hashed dim, final feature dim can more than bucket_size
        bucket_size : 1000000,
  
        # random seed
        seed : 39916801,
  
        # hashed feature name prefix
        feature_prefix : "hash_"
    },

    # preprocessing feature value
    # many learning algorithm(e.g. l1, l2 regularization) assume that features are centered around zero and have variance in the same order
    transform {
        # feature value
        switch_on : false,
        
        # preprocessing manner:
        #   standardization : x --> (x - mean) / stdvar
        #   scale_range : x --> min + (max - min) * (x - xmin) / (xmax - xmin)
        mode : "standardization",
      
        # if your preprocessing manner is "scale_range", you must supply min and max scale range
        scale_range {
            min : -1,
            max : 1
        },
	    
        # include_features and exclude_features are both empty list means doing transformation on all features.If you only want to transform a subset of features, then put related features in include_features, and set exclude_features to an empty list. If you want to tranform all features except some, then put them in exclude_features and set include_features to an empty list. If both of them are not empty list, then include_features configuration is effective and exclude_features not.
        # preprocessing only a subset of features, e.g. ["f1", "f3"]
        include_features : [],

        # preprocessing all feature except exclude_features, e.g. ["f2", "f3"]
        exclude_features : []
    }

	# if a feature's frequency of occurrence is less than filter_threshold, the feature will be filtered
    filter_threshold : 0
},

model {
    # path(checkpoint path) for model parameters, format:
    # f1,weight,precision(laplace approximate)
    # f2,weight,precision
    # ...
    # 
    # model dict data will be saved in path: "data_path" + "_dict". This dict can be used as other models "dict_path", e.g. you can use model dict of linear model with l1 as fm model's "dict_path"
    data_path : "???",
  
    # delimiter in model data
    delim : ",",
  
     # whether using user provided dict or other models' dict 
    need_dict : false,
    
    # user provided dict path, if "need_dict:false", dict_path does not need to be provided.
    # dict data format:
    # f1
    # f2
    # ....
    # 
    # attention: dict_path is input, model dict data is output
    dict_path : "",
  
    # model saved frequency
    dump_freq : 50,
  
    # whether your model uses intercept(bias)
    need_bias : true,
  
    # bias feature name
    bias_feature_name : "_bias_",
  
    # whether continue to train from checkpoint
    continue_train : false
},

loss {
    # loss function of fm model.
    # e.g. "sigmoid" stands for Logistic Regression(active function is sigmoid and loss function is negative log likelihood). "l2" stand for Linear Regression. See "Objective Function(active function and loss function)" in models.md for more details
    loss_function : "sigmoid",
  
    # except loss, if you want to evaluate other metrics, like "auc", "confusion_matrix", "RMSE", ..., see evaluation_metrics.md for more details
    evaluate_metric : ["auc"],
  
    # whether you only want to evaluate some metrics(training is finished)
    just_evaluate : false,
  
    # model parameters regularization for avoiding overfitting,
    # l1 and l2 can be used at the same time(elastic net)
    regularization : {
        # l1 regularization(parameters with the prior of laplace distribution),
        # two terms represent l1 regularization of linear part and second order interaction respectively.  
        l1 : [5.28e-9, 5.28e-9],        
        # l2 regularization(parameters the prior of guassian distribution),
        # two terms represent l2 regularization of linear part and second order interaction respectively.
        l2 : [5.28e-7, 5.28e-7]
    }
},

optimization {
    # optimizer type, this version only supports "line_search"("trust_region", "sgd" will be supported in the future)
    optimizer : "line_search",

    line_search {
        # step search stopping criterion:
        # "sufficient_decrease", "wolfe", "strong_wolfe" three modes
        # if you use "wolfe" mode, loss will always decrease no matter loss is convex or not
        mode : "wolfe",

        # backtracking is one of the most effetive step search method, just using default values
        backtracking : {
            step_decr : 0.5,
            step_incr : 2.1,
            max_iter : 55,
            min_step : 1e-16,
            max_step : 1e18,
            c1 : 1e-4,
            c2 : 0.9
        }
        
        # L-BFGS is one of the best optimization method used for convex and non-convex function,
        lbfgs {
            # L-BFGS use recent m curve infomation approximate Hessian Matrix
            m : 12,
          	
          	# converge control 
            convergence : {
                # max iter step, if you can want to converge precisely(it has risk of overfitting), increase max_iter
                max_iter : 100,
          
                # if you want to converge precisely(it has risk of overfitting), decrease eps
                eps : 1e-3
            }
        }
    }
},

hyper {
    # switch of hyperparameter optimization, if you switch on, be sure to provide test data(validation)
    switch_on : false,
  
    # between two hyper opt steps, whether previous optimized model parameters will be used for hyper opt start in the next step.
    # if restart is true, hyper opt step always uses random model parameters, otherwise use previous optimized model parameters.
    # in general, convex loss function uses restart:false, non-convex loss function uses restart:true
    restart : true,
  
    # hyperparameter optimization method:
    #   "hoag" : hyperparameter optimization with approximate gradient(modified)
    #   "grid" : grid search
    mode : "hoag",

    # hyperparameter optimization with approximate gradient. 
    # using this method, convex loss functions can always find optimal hyper parameters, non-convex loss functions can not be guaranteed.
    # attention: l1 hyperparameter is not supported in hoag method!
    hoag {
        # hyperparameters optimization inital step length, 1.0 always work
        init_step : 1.0,
  
        # between two continuous steps, if gradient is opposite, step is multiplied by step_decr_factor, step = step * step_decr_factor 
        step_decr_factor : 0.7,
        
        # if the diff of the two continuous step tests are less than a limit, hyper opt will be aborted. if you want to get more optimal hyperparameters, decrease this value.
        test_loss_reduce_limit : 1e-5,
  
        # max hyper optimization step, if you want to get more optimal hyperparameters, increase this value.
        outer_iter : 10,
        
        # l1 regularization value(fixed)
        # l1 hyperparameter is not supported in hoag method
        # two terms represent l1 regularization of linear part and second order interaction respectively.          
        l1 : [0.0, 0.0], 
  
        # l2 regularization initial value
        # two terms represent of l2 regularization of linear part and second order interaction respectively.
        l2 : [5.28e-7, 5.28e-7]
    },

    # grid search, total optimization step=l1parts * l2parts
    grid {
        # range and interval of l1 hyperparameter will be searched
        # format: [l1left, l1right, l1parts], means [left, right] interval will be divided into "l1parts" equal parts, if you want to use fixed l1 value, set as follows: [fixedvalue, fixedvalue, 1]
        # first three terms are used for linear, last three terms are used for interaction
        l1 : [1e-9, 1e-6, 5, 1e-9, 1e-6, 5], 
      
        # range and interval of l2 hyperparameter will be searched
        # format: [l2left, l2right, l2parts], means [left, right] interval will be divided into "l2parts" equal parts, if you want to use fixed l2 value, set as follows: [fixedvalue, fixedvalue, 1],
        # first three terms are used for linear, last three terms are used for interaction
        l2 : [1e-8, 1e-5, 5, 1e-8, 1e-5, 5] 
    }

},

random {
    # random initialization method, including normal/uniform distribution
    mode : "normal",
  
    # random seed
    seed : 111111,
  
    # normal distribution
    normal {
        # expectation
        mean : 0.0,
        # standard deviation
        std : 0.01
    },
	
    # uniform distribution
    uniform {
        # range start
        range_start : -0.01,
        # range end
        range_end : 0.01
    },
},

# if the value of first term >= 1, fm need linear part, if <= 0, fm doesn't have linear part
# if the value of second term >= 1, fm need second part, if <= 0, fm doesn't have second part(not recommended, it degenerates a linear model)
k : [1,8],

# whether fm has second interaction part, it indicates bias needs latent factor.
bias_need_latent_factor : false
```