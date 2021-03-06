Quick Start
***********

This is an example of a binary classification with the `adult census
<https://www.kaggle.com/wenruliu/adult-income-dataset?select=adult.csv>`__
dataset using a combination of a ``Wide`` and ``DeepDense`` model with
defaults settings.


Read and split the dataset
--------------------------

The following code snippet is not directly related to ``pytorch-widedeep``.

.. code-block:: python

    import pandas as pd
    from sklearn.model_selection import train_test_split

    df = pd.read_csv("data/adult/adult.csv.zip")
    df["income_label"] = (df["income"].apply(lambda x: ">50K" in x)).astype(int)
    df.drop("income", axis=1, inplace=True)
    df_train, df_test = train_test_split(df, test_size=0.2, stratify=df.income_label)


Prepare the wide and deep columns
---------------------------------

.. code-block:: python

    from pytorch_widedeep.preprocessing import WidePreprocessor, DensePreprocessor
    from pytorch_widedeep.models import Wide, DeepDense, WideDeep
    from pytorch_widedeep.metrics import Accuracy

    # prepare wide, crossed, embedding and continuous columns
    wide_cols = [
        "education",
        "relationship",
        "workclass",
        "occupation",
        "native-country",
        "gender",
    ]
    cross_cols = [("education", "occupation"), ("native-country", "occupation")]
    embed_cols = [
        ("education", 16),
        ("workclass", 16),
        ("occupation", 16),
        ("native-country", 32),
    ]
    cont_cols = ["age", "hours-per-week"]
    target_col = "income_label"

    # target
    target = df_train[target_col].values


Preprocessing and model components definition
---------------------------------------------

.. code-block:: python

    # wide
    preprocess_wide = WidePreprocessor(wide_cols=wide_cols, crossed_cols=cross_cols)
    X_wide = preprocess_wide.fit_transform(df_train)
    wide = Wide(wide_dim=X_wide.shape[1], pred_dim=1)

    # deepdense
    preprocess_deep = DensePreprocessor(embed_cols=embed_cols, continuous_cols=cont_cols)
    X_deep = preprocess_deep.fit_transform(df_train)
    deepdense = DeepDense(
        hidden_layers=[64, 32],
        deep_column_idx=preprocess_deep.deep_column_idx,
        embed_input=preprocess_deep.embeddings_input,
        continuous_cols=cont_cols,
    )


Build, compile, fit and predict
-------------------------------

.. code-block:: python

    # build, compile and fit
    model = WideDeep(wide=wide, deepdense=deepdense)
    model.compile(method="binary", metrics=[Accuracy])
    model.fit(
        X_wide=X_wide,
        X_deep=X_deep,
        target=target,
        n_epochs=5,
        batch_size=256,
        val_split=0.1,
    )

    # predict
    X_wide_te = preprocess_wide.transform(df_test)
    X_deep_te = preprocess_deep.transform(df_test)
    preds = model.predict(X_wide=X_wide_te, X_deep=X_deep_te)

Of course, one can do much more, such as using different initializations,
optimizers or learning rate schedulers for each component of the overall
model. Adding FC-Heads to the Text and Image components. Using the Focal Loss,
warming up individual components before joined training, etc. See the
`examples
<https://github.com/jrzaurin/pytorch-widedeep/tree/build_docs/examples>`__
directory for a better understanding of the content of the package and its
functionalities.
