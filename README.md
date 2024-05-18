# credit-card
A credit card fraud detection using tensorflow



{
  "metadata": {
    "kernelspec": {
      "language": "python",
      "display_name": "Python 3",
      "name": "python3"
    },
    "language_info": {
      "name": "python",
      "version": "3.10.12",
      "mimetype": "text/x-python",
      "codemirror_mode": {
        "name": "ipython",
        "version": 3
      },
      "pygments_lexer": "ipython3",
      "nbconvert_exporter": "python",
      "file_extension": ".py"
    },
    "colab": {
      "provenance": []
    }
  },
  "nbformat_minor": 0,
  "nbformat": 4,
  "cells": [
    {
      "cell_type": "markdown",
      "source": [
        "# Credit Card Fraud Detection with TensorFlow\n",
        "- The credit card dataset contains transaction data used for fraud detection. It includes features like time, transaction amount, and anonymized features (V1-V28).\n",
        "- The dataset has imbalanced classes, with mostly legitimate transactions (Class 0) and fewer fraudulent ones (Class 1).\n",
        "- My goal is to build a model that accurately classify transactions as fraudulent or legitimate, using techniques like regularization and evaluation metrics such as precision, recall, and F1-score."
      ],
      "metadata": {
        "id": "iN1vCNXzhuId"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import tensorflow as tf\n",
        "import numpy as np\n",
        "import pandas as pd\n",
        "import matplotlib.pyplot as plt\n",
        "import seaborn as sns\n",
        "from sklearn.model_selection import train_test_split\n",
        "from sklearn.preprocessing import StandardScaler\n",
        "from sklearn.metrics import precision_score, recall_score, f1_score\n",
        "from tensorflow.keras import regularizers\n",
        "from scipy.stats import ks_2samp\n",
        "# load data\n",
        "data = pd.read_csv(\"/content/creditcard.csv\")\n",
        "X = data.drop('Class', axis=1).values\n",
        "y = data['Class'].values\n",
        "\n",
        "# plot the class distribution\n",
        "pd.value_counts(data['Class'])"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:39:22.390432Z",
          "iopub.execute_input": "2023-11-24T17:39:22.390885Z",
          "iopub.status.idle": "2023-11-24T17:39:27.108214Z",
          "shell.execute_reply.started": "2023-11-24T17:39:22.390853Z",
          "shell.execute_reply": "2023-11-24T17:39:27.107049Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "cV4R6shchuIk",
        "outputId": "6085265b-c4ae-4128-fb6d-6a512c39496a"
      },
      "execution_count": 1,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "0    284315\n",
              "1       492\n",
              "Name: Class, dtype: int64"
            ]
          },
          "metadata": {},
          "execution_count": 1
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "data.head()"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 255
        },
        "id": "EJvZrzhbm5fF",
        "outputId": "99e794e2-dc97-43f1-d24f-9178b6b00911"
      },
      "execution_count": 2,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "   Time        V1        V2        V3        V4        V5        V6        V7  \\\n",
              "0   0.0 -1.359807 -0.072781  2.536347  1.378155 -0.338321  0.462388  0.239599   \n",
              "1   0.0  1.191857  0.266151  0.166480  0.448154  0.060018 -0.082361 -0.078803   \n",
              "2   1.0 -1.358354 -1.340163  1.773209  0.379780 -0.503198  1.800499  0.791461   \n",
              "3   1.0 -0.966272 -0.185226  1.792993 -0.863291 -0.010309  1.247203  0.237609   \n",
              "4   2.0 -1.158233  0.877737  1.548718  0.403034 -0.407193  0.095921  0.592941   \n",
              "\n",
              "         V8        V9  ...       V21       V22       V23       V24       V25  \\\n",
              "0  0.098698  0.363787  ... -0.018307  0.277838 -0.110474  0.066928  0.128539   \n",
              "1  0.085102 -0.255425  ... -0.225775 -0.638672  0.101288 -0.339846  0.167170   \n",
              "2  0.247676 -1.514654  ...  0.247998  0.771679  0.909412 -0.689281 -0.327642   \n",
              "3  0.377436 -1.387024  ... -0.108300  0.005274 -0.190321 -1.175575  0.647376   \n",
              "4 -0.270533  0.817739  ... -0.009431  0.798278 -0.137458  0.141267 -0.206010   \n",
              "\n",
              "        V26       V27       V28  Amount  Class  \n",
              "0 -0.189115  0.133558 -0.021053  149.62      0  \n",
              "1  0.125895 -0.008983  0.014724    2.69      0  \n",
              "2 -0.139097 -0.055353 -0.059752  378.66      0  \n",
              "3 -0.221929  0.062723  0.061458  123.50      0  \n",
              "4  0.502292  0.219422  0.215153   69.99      0  \n",
              "\n",
              "[5 rows x 31 columns]"
            ],
            "text/html": [
              "\n",
              "  <div id=\"df-441e9be1-cef4-42c4-abb1-080198f7da01\" class=\"colab-df-container\">\n",
              "    <div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>Time</th>\n",
              "      <th>V1</th>\n",
              "      <th>V2</th>\n",
              "      <th>V3</th>\n",
              "      <th>V4</th>\n",
              "      <th>V5</th>\n",
              "      <th>V6</th>\n",
              "      <th>V7</th>\n",
              "      <th>V8</th>\n",
              "      <th>V9</th>\n",
              "      <th>...</th>\n",
              "      <th>V21</th>\n",
              "      <th>V22</th>\n",
              "      <th>V23</th>\n",
              "      <th>V24</th>\n",
              "      <th>V25</th>\n",
              "      <th>V26</th>\n",
              "      <th>V27</th>\n",
              "      <th>V28</th>\n",
              "      <th>Amount</th>\n",
              "      <th>Class</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>0.0</td>\n",
              "      <td>-1.359807</td>\n",
              "      <td>-0.072781</td>\n",
              "      <td>2.536347</td>\n",
              "      <td>1.378155</td>\n",
              "      <td>-0.338321</td>\n",
              "      <td>0.462388</td>\n",
              "      <td>0.239599</td>\n",
              "      <td>0.098698</td>\n",
              "      <td>0.363787</td>\n",
              "      <td>...</td>\n",
              "      <td>-0.018307</td>\n",
              "      <td>0.277838</td>\n",
              "      <td>-0.110474</td>\n",
              "      <td>0.066928</td>\n",
              "      <td>0.128539</td>\n",
              "      <td>-0.189115</td>\n",
              "      <td>0.133558</td>\n",
              "      <td>-0.021053</td>\n",
              "      <td>149.62</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>0.0</td>\n",
              "      <td>1.191857</td>\n",
              "      <td>0.266151</td>\n",
              "      <td>0.166480</td>\n",
              "      <td>0.448154</td>\n",
              "      <td>0.060018</td>\n",
              "      <td>-0.082361</td>\n",
              "      <td>-0.078803</td>\n",
              "      <td>0.085102</td>\n",
              "      <td>-0.255425</td>\n",
              "      <td>...</td>\n",
              "      <td>-0.225775</td>\n",
              "      <td>-0.638672</td>\n",
              "      <td>0.101288</td>\n",
              "      <td>-0.339846</td>\n",
              "      <td>0.167170</td>\n",
              "      <td>0.125895</td>\n",
              "      <td>-0.008983</td>\n",
              "      <td>0.014724</td>\n",
              "      <td>2.69</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>1.0</td>\n",
              "      <td>-1.358354</td>\n",
              "      <td>-1.340163</td>\n",
              "      <td>1.773209</td>\n",
              "      <td>0.379780</td>\n",
              "      <td>-0.503198</td>\n",
              "      <td>1.800499</td>\n",
              "      <td>0.791461</td>\n",
              "      <td>0.247676</td>\n",
              "      <td>-1.514654</td>\n",
              "      <td>...</td>\n",
              "      <td>0.247998</td>\n",
              "      <td>0.771679</td>\n",
              "      <td>0.909412</td>\n",
              "      <td>-0.689281</td>\n",
              "      <td>-0.327642</td>\n",
              "      <td>-0.139097</td>\n",
              "      <td>-0.055353</td>\n",
              "      <td>-0.059752</td>\n",
              "      <td>378.66</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>1.0</td>\n",
              "      <td>-0.966272</td>\n",
              "      <td>-0.185226</td>\n",
              "      <td>1.792993</td>\n",
              "      <td>-0.863291</td>\n",
              "      <td>-0.010309</td>\n",
              "      <td>1.247203</td>\n",
              "      <td>0.237609</td>\n",
              "      <td>0.377436</td>\n",
              "      <td>-1.387024</td>\n",
              "      <td>...</td>\n",
              "      <td>-0.108300</td>\n",
              "      <td>0.005274</td>\n",
              "      <td>-0.190321</td>\n",
              "      <td>-1.175575</td>\n",
              "      <td>0.647376</td>\n",
              "      <td>-0.221929</td>\n",
              "      <td>0.062723</td>\n",
              "      <td>0.061458</td>\n",
              "      <td>123.50</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>2.0</td>\n",
              "      <td>-1.158233</td>\n",
              "      <td>0.877737</td>\n",
              "      <td>1.548718</td>\n",
              "      <td>0.403034</td>\n",
              "      <td>-0.407193</td>\n",
              "      <td>0.095921</td>\n",
              "      <td>0.592941</td>\n",
              "      <td>-0.270533</td>\n",
              "      <td>0.817739</td>\n",
              "      <td>...</td>\n",
              "      <td>-0.009431</td>\n",
              "      <td>0.798278</td>\n",
              "      <td>-0.137458</td>\n",
              "      <td>0.141267</td>\n",
              "      <td>-0.206010</td>\n",
              "      <td>0.502292</td>\n",
              "      <td>0.219422</td>\n",
              "      <td>0.215153</td>\n",
              "      <td>69.99</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "<p>5 rows × 31 columns</p>\n",
              "</div>\n",
              "    <div class=\"colab-df-buttons\">\n",
              "\n",
              "  <div class=\"colab-df-container\">\n",
              "    <button class=\"colab-df-convert\" onclick=\"convertToInteractive('df-441e9be1-cef4-42c4-abb1-080198f7da01')\"\n",
              "            title=\"Convert this dataframe to an interactive table.\"\n",
              "            style=\"display:none;\">\n",
              "\n",
              "  <svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\" viewBox=\"0 -960 960 960\">\n",
              "    <path d=\"M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z\"/>\n",
              "  </svg>\n",
              "    </button>\n",
              "\n",
              "  <style>\n",
              "    .colab-df-container {\n",
              "      display:flex;\n",
              "      gap: 12px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert {\n",
              "      background-color: #E8F0FE;\n",
              "      border: none;\n",
              "      border-radius: 50%;\n",
              "      cursor: pointer;\n",
              "      display: none;\n",
              "      fill: #1967D2;\n",
              "      height: 32px;\n",
              "      padding: 0 0 0 0;\n",
              "      width: 32px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert:hover {\n",
              "      background-color: #E2EBFA;\n",
              "      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "      fill: #174EA6;\n",
              "    }\n",
              "\n",
              "    .colab-df-buttons div {\n",
              "      margin-bottom: 4px;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert {\n",
              "      background-color: #3B4455;\n",
              "      fill: #D2E3FC;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert:hover {\n",
              "      background-color: #434B5C;\n",
              "      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);\n",
              "      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));\n",
              "      fill: #FFFFFF;\n",
              "    }\n",
              "  </style>\n",
              "\n",
              "    <script>\n",
              "      const buttonEl =\n",
              "        document.querySelector('#df-441e9be1-cef4-42c4-abb1-080198f7da01 button.colab-df-convert');\n",
              "      buttonEl.style.display =\n",
              "        google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "\n",
              "      async function convertToInteractive(key) {\n",
              "        const element = document.querySelector('#df-441e9be1-cef4-42c4-abb1-080198f7da01');\n",
              "        const dataTable =\n",
              "          await google.colab.kernel.invokeFunction('convertToInteractive',\n",
              "                                                    [key], {});\n",
              "        if (!dataTable) return;\n",
              "\n",
              "        const docLinkHtml = 'Like what you see? Visit the ' +\n",
              "          '<a target=\"_blank\" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'\n",
              "          + ' to learn more about interactive tables.';\n",
              "        element.innerHTML = '';\n",
              "        dataTable['output_type'] = 'display_data';\n",
              "        await google.colab.output.renderOutput(dataTable, element);\n",
              "        const docLink = document.createElement('div');\n",
              "        docLink.innerHTML = docLinkHtml;\n",
              "        element.appendChild(docLink);\n",
              "      }\n",
              "    </script>\n",
              "  </div>\n",
              "\n",
              "\n",
              "<div id=\"df-b50761f6-9a10-4e8f-9d55-b5506ae3d441\">\n",
              "  <button class=\"colab-df-quickchart\" onclick=\"quickchart('df-b50761f6-9a10-4e8f-9d55-b5506ae3d441')\"\n",
              "            title=\"Suggest charts\"\n",
              "            style=\"display:none;\">\n",
              "\n",
              "<svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\"viewBox=\"0 0 24 24\"\n",
              "     width=\"24px\">\n",
              "    <g>\n",
              "        <path d=\"M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z\"/>\n",
              "    </g>\n",
              "</svg>\n",
              "  </button>\n",
              "\n",
              "<style>\n",
              "  .colab-df-quickchart {\n",
              "      --bg-color: #E8F0FE;\n",
              "      --fill-color: #1967D2;\n",
              "      --hover-bg-color: #E2EBFA;\n",
              "      --hover-fill-color: #174EA6;\n",
              "      --disabled-fill-color: #AAA;\n",
              "      --disabled-bg-color: #DDD;\n",
              "  }\n",
              "\n",
              "  [theme=dark] .colab-df-quickchart {\n",
              "      --bg-color: #3B4455;\n",
              "      --fill-color: #D2E3FC;\n",
              "      --hover-bg-color: #434B5C;\n",
              "      --hover-fill-color: #FFFFFF;\n",
              "      --disabled-bg-color: #3B4455;\n",
              "      --disabled-fill-color: #666;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart {\n",
              "    background-color: var(--bg-color);\n",
              "    border: none;\n",
              "    border-radius: 50%;\n",
              "    cursor: pointer;\n",
              "    display: none;\n",
              "    fill: var(--fill-color);\n",
              "    height: 32px;\n",
              "    padding: 0;\n",
              "    width: 32px;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart:hover {\n",
              "    background-color: var(--hover-bg-color);\n",
              "    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "    fill: var(--button-hover-fill-color);\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart-complete:disabled,\n",
              "  .colab-df-quickchart-complete:disabled:hover {\n",
              "    background-color: var(--disabled-bg-color);\n",
              "    fill: var(--disabled-fill-color);\n",
              "    box-shadow: none;\n",
              "  }\n",
              "\n",
              "  .colab-df-spinner {\n",
              "    border: 2px solid var(--fill-color);\n",
              "    border-color: transparent;\n",
              "    border-bottom-color: var(--fill-color);\n",
              "    animation:\n",
              "      spin 1s steps(1) infinite;\n",
              "  }\n",
              "\n",
              "  @keyframes spin {\n",
              "    0% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "      border-left-color: var(--fill-color);\n",
              "    }\n",
              "    20% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    30% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    40% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    60% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    80% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "    90% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "  }\n",
              "</style>\n",
              "\n",
              "  <script>\n",
              "    async function quickchart(key) {\n",
              "      const quickchartButtonEl =\n",
              "        document.querySelector('#' + key + ' button');\n",
              "      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.\n",
              "      quickchartButtonEl.classList.add('colab-df-spinner');\n",
              "      try {\n",
              "        const charts = await google.colab.kernel.invokeFunction(\n",
              "            'suggestCharts', [key], {});\n",
              "      } catch (error) {\n",
              "        console.error('Error during call to suggestCharts:', error);\n",
              "      }\n",
              "      quickchartButtonEl.classList.remove('colab-df-spinner');\n",
              "      quickchartButtonEl.classList.add('colab-df-quickchart-complete');\n",
              "    }\n",
              "    (() => {\n",
              "      let quickchartButtonEl =\n",
              "        document.querySelector('#df-b50761f6-9a10-4e8f-9d55-b5506ae3d441 button');\n",
              "      quickchartButtonEl.style.display =\n",
              "        google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "    })();\n",
              "  </script>\n",
              "</div>\n",
              "    </div>\n",
              "  </div>\n"
            ]
          },
          "metadata": {},
          "execution_count": 2
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "data = data.dropna(how='any',axis=0)"
      ],
      "metadata": {
        "id": "5S4d91uXnCB9"
      },
      "execution_count": 3,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "data.isna().sum()"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "MUqlqJvDnOHw",
        "outputId": "cacc6e4c-9499-47a7-8ff5-748fefac3f68"
      },
      "execution_count": 4,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "Time      0\n",
              "V1        0\n",
              "V2        0\n",
              "V3        0\n",
              "V4        0\n",
              "V5        0\n",
              "V6        0\n",
              "V7        0\n",
              "V8        0\n",
              "V9        0\n",
              "V10       0\n",
              "V11       0\n",
              "V12       0\n",
              "V13       0\n",
              "V14       0\n",
              "V15       0\n",
              "V16       0\n",
              "V17       0\n",
              "V18       0\n",
              "V19       0\n",
              "V20       0\n",
              "V21       0\n",
              "V22       0\n",
              "V23       0\n",
              "V24       0\n",
              "V25       0\n",
              "V26       0\n",
              "V27       0\n",
              "V28       0\n",
              "Amount    0\n",
              "Class     0\n",
              "dtype: int64"
            ]
          },
          "metadata": {},
          "execution_count": 4
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "data.replace([np.inf, -np.inf], np.nan, inplace=True)"
      ],
      "metadata": {
        "id": "BQzOfxgcn4HT"
      },
      "execution_count": 5,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Distribution of credit card transactions over time"
      ],
      "metadata": {
        "id": "ATWL0xHvhuIp"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# Set the aesthetic style of the plots\n",
        "sns.set_style(\"whitegrid\")\n",
        "\n",
        "f, (ax1, ax2) = plt.subplots(2, 1, sharex=True, figsize=(12, 6))\n",
        "\n",
        "bins = 100\n",
        "\n",
        "# Plotting for Fraud transactions\n",
        "ax1.hist(data.Time[data.Class == 1], bins=bins, color='red', alpha=0.7)\n",
        "ax1.set_title('Fraud', fontsize=14)\n",
        "ax1.set_ylabel('Number of Transactions', fontsize=12)\n",
        "ax1.grid(True, linestyle='--', alpha=0.5)\n",
        "\n",
        "# Plotting for Normal transactions\n",
        "ax2.hist(data.Time[data.Class == 0], bins=bins, color='blue', alpha=0.7)\n",
        "ax2.set_title('Normal', fontsize=14)\n",
        "ax2.set_xlabel('Time (in Seconds)', fontsize=12)\n",
        "ax2.set_ylabel('Number of Transactions', fontsize=12)\n",
        "ax2.grid(True, linestyle='--', alpha=0.5)\n",
        "\n",
        "# Remove top and right spines\n",
        "for ax in [ax1, ax2]:\n",
        "    ax.spines['top'].set_visible(False)\n",
        "    ax.spines['right'].set_visible(False)\n",
        "\n",
        "plt.tight_layout()\n",
        "plt.show()"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:40:02.077073Z",
          "iopub.execute_input": "2023-11-24T17:40:02.077473Z",
          "iopub.status.idle": "2023-11-24T17:40:03.297687Z",
          "shell.execute_reply.started": "2023-11-24T17:40:02.077442Z",
          "shell.execute_reply": "2023-11-24T17:40:03.296570Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 569
        },
        "id": "nWHhmMbZhuIq",
        "outputId": "91ae54f6-34a1-4df5-9dc6-8958ebf46ae7"
      },
      "execution_count": 6,
      "outputs": [
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 1200x600 with 2 Axes>"
            ],
            "image/png": "iVBORw0KGgoAAAANSUhEUgAABKUAAAJOCAYAAABm7rQwAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjcuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/bCgiHAAAACXBIWXMAAA9hAAAPYQGoP6dpAADn00lEQVR4nOzdeXwU9f0/8Ndn9kpCLsgGEsATBQ9EsCpiUbzvohwKoqB9UG+0Wtt61P5Aa4H6xapfEZWWarUqWu+j1qO2VCuK2uJRKfX6KhATsgFCloQ9P78/NjPsmWz2s7szs3k9Hw8eJLOTnc97P6/9zOST2RkhpZQgIiIiIiIiIiIqIs3sBhARERERERERUf/DSSkiIiIiIiIiIio6TkoREREREREREVHRcVKKiIiIiIiIiIiKjpNSRERERERERERUdJyUIiIiIiIiIiKiouOkFBERERERERERFR0npYiIiIiIiIiIqOg4KUVEREREREREREXHSSkiIiIi6tWoUaMwe/Zss5tBREREJcRpdgOIiIiISt3GjRtx/PHH97jOe++9h+rq6iK1iIiIiMh8nJQiIiIiKpLdd98dkydPTvuYx+MpcmuIiIiIzMVJKSIiIqIi2X333XHllVea3QwiIiIiS+A1pYiIiIgs4Omnn8aoUaPw9NNP44033sDMmTMxbtw4HHfccQCAYDCIhx9+GHPnzsWkSZMwevRoTJgwAfPmzcOnn36a8nx33303Ro0ahXfffbfHbSX74x//iDPOOAMHHXQQJk2ahNtuuw2BQCD/BRMREVG/xzOliIiIiCzkz3/+M/7xj3/gmGOOwaxZs+D3+wEA7e3tWLhwIQ499FBMmjQJ1dXV2LBhA9544w38/e9/xx/+8AeMGTNGadv33HMP/vd//xderxfnnHMOnE4nXn75ZXz55Zf5KI2IiIgoASeliIiIiIrkm2++wd13352y/KijjjK+fvPNN7FixQoceeSRCevU1NTgb3/7G4YMGZKw/LPPPsM555yDO+64Aw888EDObfv666+xbNkyDBkyBM888wzq6uoAAFdeeSWmT5+e8/MSERERZcJJKSIiIqIi+eabb7B06dKU5VVVVcad944//viUCSkAcLvdKRNSALDvvvti/PjxeOuttxAKheByuXJq2wsvvIBwOIzvf//7xoQUAFRWVuKyyy7DT3/605yel4iIiCgTTkoRERERFcnEiROxYsWKtI/p13fq6SN469atw29/+1t88MEH8Pl8CIVCCY9v3boVgwcPzqlt69evBwB85zvfSXns0EMPzek5iYiIiHrCSSkiIiIiC4k/SyneP//5T1xwwQUAgO9+97vYc889UVFRASEEXn/9dfznP/9BMBjMebsdHR0Zt+/1enN+XiIiIqJMOClFREREZCFCiLTL77vvPgSDQTzyyCMpZy6tXbs24/NEIpGUx/QJqHhVVVUAgLa2NgwbNizhMZ/Pl1XbiYiIiPpCM7sBRERERNS7b775BrW1tSkTUl1dXfj0009T1q+pqQEAtLS0pDy2bt26lGWjRo0CAHzwwQcpj73//vs5tZmIiIioJ5yUIiIiIrKBYcOGob29HZ999pmxLBKJ4Fe/+hW2bNmSsv5BBx0EAHj22WcRjUaN5f/617/wwgsvpKz/ve99Dw6HAw888ADa2tqM5X6/H/fee28+SyEiIiICwI/vEREREdnC+eefj7feeguzZs3CqaeeCrfbjTVr1qClpQWHH3441qxZk7D+2LFjccghh+Cdd97BjBkzcOihh6KpqQl/+ctfcOyxx+K1115LWH+PPfbA5ZdfjrvvvhuTJ0/GqaeeCofDgVdffRWjRo3CV199VcxyiYiIqB/gmVJERERENnDsscfif//3f7Hbbrvh+eefx4svvoi9994bTz75ZMo1oHTLli3DWWedhW+++QaPPvoompubcd999+G4445Lu/68efNw6623ora2FitXrsSf//xnnHLKKbjzzjsLWBkRERH1V0JKKc1uBBERERERERER9S88U4qIiIiIiIiIiIqOk1JERERERERERFR0nJQiIiIiIiIiIqKi46QUEREREREREREVHSeliIiIiIiIiIio6DgpRURERERERERERcdJqRxIKRGJRCClNLspRERERERERES2xEmpHESjUaxduxbRaNTspiiRUqKjo4OTa5QzZohUMUOkihkiVcwQqWKGSBUzRKrsnCFOSvVjdg4uWQMzRKqYIVLFDJEqZohUMUOkihkiVXbOECeliIiIiIiIiIio6Dgp1c95PB6zm0A2xwyRKmaIVDFDpIoZIlXMEKlihkiVXTMkpB3P7zJZJBLB2rVrMXbsWDgcDrObQ0RERERERERkOzxTqh+z8+dOyRqYIVLFDJEqZohUMUOkihki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fvvth7lz56Ktrc3sphWNlBLbtm2z5RX6yRokgFAoxDuFUM6YIVIlAYR45zRSwHGIVDFDpIp33yNVdv7dvt9OSj3wwAM455xzMG3aNOyzzz64+eabUVZWhqeeesrsphWNlBKdnZ22DC5ZRPddr8AMUa6YIVIlJaLRqNmtIDvjOESqmCFS1X33PWaIcmXn3+375aRUMBjEv//9bxx55JHGMk3TcOSRR+Jf//qXiS0jIiIiIiIiIuof+uXd97Zu3YpIJIK6urqE5XV1dfjyyy97/Xl99jEUChl/nRXdF6eT+ix3N3158l9xc1kev+3elmualtKW5OXRaBSRSAShUAhOpxMyEgEaGgCnU39yCHSfRqo/j9cLRKMQkUivbRf683V/xj5l1tbrjT1v/Da7t4vkbUoJ0dAA6XKl/AVBCBFrY9LzGcvj1xcCwuuNtSVdrfHrxm/XmfRWiV8/frsuV2Lbe6k1pY251Kq/vqq19tCviFs/vl8hJSKhEMJx11EQQkCmqbdPtcZnL12tya9vDv2akrvu9aG3MW494XKlvr59qbV7u0JKyHC4xzGi1/dND7Uabc+i1oQ25lprujEivtY07+3k941sbEQEQER/7yTVmraGDLWm3WZfa9XbmKnWNO/tXmvtXi9hnEjTr+lq7TFLFqoV8c/dS619GrN7e3272xmpr0ckvtZ075s+1lr0/VMOWeprrQltj681GoUMhRIWpzuOEJFIcfdPWdQKAKirS6ihx2Oj5GOdbgn7snzvn+rqIJNe46Ic76WpNe2+uHv/n69j2HTHk4jfZvJrl2kcy+Z9A8SylOH1zZjhPhzXZerX+PWk04lIKBTblwlRuDHbjGNYs/ZPearVqCmLWnM6ruth/5S21gxjdrS+PrZeYyNEd44sdwxrxv4pTb2mH8NmWSuA2P4pEslu/5RpLM9yuf77fTgcTriulNnzEULEbkjT07WuhLTj+V2KWlpacPTRR2PlypUYN26csfy2227De++9hz/+8Y89/nwwGMTHH39c6GYSEREREREREdnW2LFjY3cozaBfnik1cOBAOByOlIuat7W1wev19vrzTqcTBx10UK8zfkRERERERERE/ZWm9XzVqH45KeV2u3HggQdi9erVOOGEEwAA0WgUq1evxvnnn9/rz2uaBrfbXehmEhERERERERGVrH45KQUA3//+93Hddddh9OjRGDNmDH7/+9+jq6sLU6dONbtpREREREREREQlr99OSp122mnYsmUL/vd//xetra3Yf//98dvf/jarj+8REREREREREZGafnmhcyIiIiIiIiIiMlfPV5wiIiIiIiIiIiIqAE5KERERERERERFR0XFSioiIiIiIiIiIio6TUkREREREREREVHSclCIiIiIiIiIioqLjpBQRERERERERERUdJ6WIiIiIiIiIiKjoOClFRERERERERERFx0kpIiIiIiIiIiIqOk5KERERERERERFR0XFSioiIiIiIiIiIio6TUkREREREREREVHSclCIiIiIiIiIioqLjpBQRERERERERERUdJ6WIiIiIiIiIiKjoOClFRERERERERERFx0kpIiIiIiIiIiIqOk5KEREREZGSjRs3YtSoUbj++uvNbgoRERHZCCeliIiIiBTpkzKjRo3C3Llz066zdu1aTtwQERERxeGkFBEREVEevfXWW1i9erXZzSAiIiKyPE5KEREREeXJsGHDoGkalixZAiml2c0hIiIisjROShERERHlyV577YUzzzwTn3zyCV5++eWsfmbTpk248cYbcdRRR2H06NE4+uijceONN6KpqSll3dmzZ2PUqFEIBAK44447cMIJJ+DAAw/E3XffDQAYNWoUZs+ejZaWFlx77bUYP348xo0bh4svvhgbNmwAAHzxxRe4/PLLcfjhh2PcuHG46qqr4PP5Urb15JNP4rLLLsNxxx2Hgw46CIcffjjmzp2Ld955R+EVIiIiItqFk1JEREREeXTVVVfB7XbjzjvvRCgU6nHdr776CtOnT8dTTz2FAw88EN///vdxwAEH4KmnnsK0adPw1Vdfpf25K6+8Es888wzGjx+POXPmYPjw4cZj7e3tOPfcc7Fx40ZMmTIF48ePx6pVq/D9738f//3vfzFz5kx0dnZi2rRpGD16NF555RX86Ec/StnGLbfcgra2NkyYMAEXXnghjjnmGPzrX//C97//fbz++utqLxIRERERAKfZDSAiIiIqJUOHDsX555+P3/3ud3j88cdx/vnnZ1x3/vz52LJlC2655RbMmDHDWP7II4/glltuwYIFC/D73/8+5ec2b96M559/HrW1tSmPrV+/HhdeeCFuuOEGY9mCBQvw2GOP4bzzzsO8efNwwQUXAACklLjkkkuwatUq/Pvf/8aBBx5o/MxLL72E3XbbLWW706ZNw//8z//ghBNOyPo1ISIiIkqHZ0oRERER5dmll16K6upqLFu2DDt27Ei7TlNTE959913ss88+OOeccxIeO/fcc7H33nvjnXfewbfffpvys1deeWXaCSkAqKiowNVXX52w7IwzzgAA1NbWYs6cOcZyIQROO+00AMB//vOfhJ9JnpACgMGDB+Pkk0/G//3f/2HTpk1pt09ERESULU5KEREREeVZTU0NLrroIrS1teF3v/td2nXWrVsHADjssMMghEh4TNM0HHbYYQnrxRszZkzGbe+5554oLy9PWFZfXw8gds2p5G0NHjwYQOwsqHgbNmzATTfdhBNOOAEHHXQQRo0ahVGjRuHhhx9Ouz4RERFRX/Hje0REREQFMGfOHDzyyCP43e9+h1mzZqU87vf7AQBerzftz+sTSfp68TL9DABUVlamLHM4HL0+Fg6HjWVff/01zj77bPj9fowfPx7HHnssKisroWka1qxZgzVr1iAYDGZsAxEREVE2OClFREREVABlZWW48sor8bOf/QxLly7FmWeemfC4PkGU7s53ANDa2pqwXrzks53y7cEHH0R7eztuu+22lHb/v//3/7BmzZqCbp+IiIj6B358j4iIiKhApkyZgn333Rd//OMf8fXXXyc8tv/++wMA3n//fUgpEx6TUuL9999PWK+YvvnmGwDA8ccfn9Kuf/3rX0VvDxEREZUmTkoRERERFYjD4cA111yDUCiEpUuXJjw2dOhQjB8/Hp999hmefPLJhMcef/xxfPHFFzjiiCPQ2NhYzCYDAIYNGwYA+OCDDxKWL1++HP/973+L3h4iIiIqTfz4HhEREVEBHX/88fjOd76TMsEDAAsWLMCsWbPw85//HH/961+xzz774LPPPsMbb7yBQYMGYcGCBcVvMICZM2fi6aefxlVXXYVTTz0VtbW1WLt2LT799FMcc8wx+Nvf/mZKu4iIiKi08EwpIiIiogL78Y9/nHb53nvvjaeeegpTpkzBRx99hBUrVuDjjz/G1KlT8eSTT2KvvfYqcktjDjjgAKxYsQIHHHAAXn31VTz11FOorq7GY489htGjR5vSJiIiIio9QiZfxICIiIiIiIiIiKjAeKYUEREREREREREVHSeliIiIiIiIiIio6DgpRURERERERERERcdJKSIiIiIiIiIiKjpOShERERER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SrGfrFyTlBKRSAQASqYmoPT6yco17dixwxiHqqurc6opFAohGJQIBGJ1ud1uCCESniMYFADcCIfD7KcSq8nr9SIcDqOlpcU4eLZ7TaXYT1auST+m1n8pLIWaSrGfrFxTW1tbwjF1KdRUiv1k5ZrKy8sBANu3b0dXV5dlaspmfkXIdH8u6oWUEo8//jiefPJJbNiwAdu3b099YiHw6aef9vWpE2zfvh3HHnssrr/+epSVleGGG27AJ598krDO9OnTMX78ePzkJz/Bz3/+czQ1NWHFihXG411dXRg7diyWL1+OSZMm4eSTT8bUqVNxySWXGOusWrUKF198MT788MOsXrRIJIK1a9dizJgxxjWv7DjbGo1G0dLSgiFDhsDpdPaLGWTWlN+aotEompubMWTIEGOC1u41pVvOmgp7plRLSwsaGxuRzK419bacNeW3pkgkYuzLHA5HTjVt3CixYAGgH2fpf9SLb0pDA7BgATBsGPup1GoCgG+//TZhX2b3mkqxn6xck74va2hoMNpj95p6W86a8luTPjGuj0OlUFMp9pOVa5JSGhnSt2OFmuLbkklOZ0rddtttePDBB7H//vtj8uTJqKmpyeVpelVdXY0999wT33zzDY488kiEQiFs37494WyptrY24xpUXq8XH330UcJz6Hfni18n+Y59Pp8PlZWVfTpLCoi90Mmf19Q7Md26mZ6jL8szdWq65Znaoi8XQqCuri5lYi3b57FiTarLWVPfatI0zchQusFPte3sp9KvSR+HMj1HprZnWm6FmrJZzpryV5PD4UgZh3JpuxC7JqN2LU/8etc/9lMp1SSlTLsv66ntVq8pn8tZU+816fuy+MmEQrWd/VSaNaXbl/W0vh1qKsV+snJNUkoMGjTIGIdU256vmrKR06TUs88+i5NOOgl33XVXLj+etR07dmDDhg2or6/H6NGj4XK5sHr1apx88skAgC+//BJNTU0YO3YsAGDs2LG477770NbWhrq6OgDA22+/jcrKSuyzzz7GOn//+98TtvP2228bz9GfCCH6PBFHFI8ZIlXMEKlihkgVM0SqmCFSxQyRKjtnKKfLsu/cuTPhguP58qtf/Qpr1qzBxo0b8c9//hPz5s2Dpmk444wzUFVVhWnTpmHx4sV455138Mknn+DGG2/EuHHjjAmliRMnYp999sFPf/pT/Oc//8Gbb76J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8Msvv6jS3HLLLejYsaPq39y5c1Vpjh8/jokTJyIjIwO9e/fGww8/DJfLpUrz2WefYciQIUhPT8fVV1+NzZs3B+SHMRh9/vnPfwbEx4ABA5TXKyoqsGDBAmRlZaFHjx648847UVhYqNoH46dx69+/f0AMdezYEQsWLADANogCffHFF7j99tuRk5ODjh07Ytu2barXhRB48sknkZOTg27dumHs2LH49ddfVWlOnz6NmTNnIjMzE7169cIDDzyAM2fOqNJ8//33uPnmm9G1a1dcfvnlWLNmTUBetm7digEDBqBr164YOHAgduzYEXZeqO5VFUNOpxOPPPIIBg4ciO7duyMnJwf33Xcf8vPzVfvQartWr16tSsMYariqa4dmz54dEB/jx49XpWE71LhVF0Na10YdO3bEs88+q6RpsO2QoEbpvffeE126dBGvv/66+Omnn8ScOXNEr169RGFhYaSzRrVo3LhxYtOmTeLHH38UeXl5YsKECaJfv37izJkzSpoxY8aIOXPmiN9//135V1paqrzucrnEDTfcIMaOHSu+++47sX37dpGVlSUeffRRJc3hw4dFRkaGWLJkiTh48KB48cUXRefOncXHH3+spGEMRqennnpK5ObmquKjqKhIeX3u3Lni8ssvFzt37hTffPONuPHGG8XIkSOV1xk/VFRUpIqfTz/9VKSlpYndu3cLIdgGUaDt27eLxx57THzwwQciLS1NfPjhh6rXV61aJXr27Ck+/PBDkZeXJ26//XbRv39/8ccffyhpxo8fLwYNGiT27dsnvvjiC3H11VeLu+++W3m9tLRUZGdni5kzZ4off/xRvPvuu6Jbt25i48aNSpovv/xSdO7cWaxZs0YcPHhQPP7446JLly7ihx9+CCsvVPeqiqGSkhIxduxY8d5774mff/5Z7N27VwwfPlwMGTJEtY8rrrhCrFixQtU2+V4/MYYaturaoVmzZonx48er4uP06dOqNGyHGrfqYsg3dn7//Xfx+uuvi44dO4rDhw8raRpqO8RBqUZq+PDhYsGCBcrvbrdb5OTkiFWrVkUwV1TXioqKRFpamvj888+VbWPGjBGLFi0K+p7t27eLTp06iYKCAmXbhg0bRGZmpqioqBBCCLFs2TKRm5uret+MGTPEuHHjlN8Zg9HpqaeeEoMGDdJ8raSkRHTp0kVs3bpV2Xbw4EGRlpYm9u7dK4Rg/FCgRYsWiauuukp4PB4hBNsgqpr/hbzH4xF9+vQRzz77rLKtpKREpKeni3fffVcIUdkOff3110qaHTt2iI4dO4oTJ04IIYR4+eWXxcUXX6zEkBBCPPLII+Laa69Vfp8+fbqYOHGiKj8jRowQ//jHP0LOC0We1odBf/v37xdpaWni2LFjyrYrrrhCrFu3Luh7GEONR7BBqcmTJwd9D9sh8hVKOzR58mRx6623qrY11HaIj+81Qg6HA99++y2ys7OVbSaTCdnZ2di7d28Ec0Z1rbS0FACQkJCg2v7OO+8gKysLN9xwAx599FGUl5crr+3btw9paWlITk5WtuXk5KCsrAwHDx5U0vTu3Vu1z5ycHOzbtw8AYzDa/fbbb8jJycGVV16JmTNn4vjx4wCAAwcOwOl0quq1ffv2aN26tVL3jB/y5XA48Pbbb2PYsGGQJEnZzjaIQnX06FEUFBSo6jI+Ph4ZGRlKXe7duxfNmjVD165dlTTZ2dkwmUzKI5v79u1Dr169YLPZlDQ5OTk4dOgQiouLlTRVxVUoeaHoUFZWBkmS0KxZM9X2NWvWICsrC4MHD8azzz6remyYMUSff/45evfujWuvvRbz5s3DqVOnlNfYDlE4CgsLsWPHDgwfPjzgtYbYDllqZa9Ur506dQputxtJSUmq7UlJSQHzC1HD5fF4sHjxYmRmZiItLU3ZfsMNN6B169Zo2bIlfvjhByxfvhyHDh3CihUrAHgbSd8PgwCU3wsKCqpMU1ZWhj/++APFxcWMwSjVrVs3LFmyBBdccAEKCgrw9NNPY/To0XjnnXdQWFgIq9UacBGflJRUbWwAjJ/GaNu2bSgtLcWQIUOUbWyDKBxynWvVpTyfXWFhIVq0aKF63WKxICEhQRUzbdu2VaWRY6iwsBAJCQmaceV7nFDyQvVfRUUFli9fjtzcXMTFxSnbb7nlFlx00UVISEjA3r178dhjj6GgoAD3338/AMZQY9e3b19cffXVaNu2LY4cOYLHHnsMEyZMwCuvvAKz2cx2iMLyxhtvoGnTprjmmmtU2xtqO8RBKaJGasGCBfjpp5+wYcMG1faRI0cqP3fs2BEpKSkYO3YsDh8+jPPPP7+us0n1zOWXX6783KlTJ2RkZOCKK67A1q1bERMTE8GcUTTatGkTLrvsMqSmpirb2AYRUaQ4nU5Mnz4dQghl8QXZbbfdpvzcqVMnWK1WzJs3DzNnzlTdlUCNU25urvKzPAH1VVddpdw9RRSOTZs2YeDAgbDb7artDbUd4uN7jVDz5s1hNptRVFSk2l5UVBQwakoN08KFC7F9+3a88MILaNWqVZVpMzIyAHgf2QK8o+3+o+Ty7ykpKVWmiYuLQ0xMDGOwAWnWrBn+/Oc/4/Dhw0hOTobT6URJSYkqTVFRUbWxATB+Gptjx45h586dmrem+2IbRFWR67yqukxOTsbJkydVr7tcLhQXF4fUNvnuxz+N73FCyQvVX06nEzNmzMDx48fx3HPPqe6S0pKRkQGXy4WjR48CYAyRWrt27dC8eXNV38V2iEKxZ88eHDp0CCNGjKg2bUNphzgo1QjZbDZ06dIFu3btUrZ5PB7s2rULPXr0iGDOqLYJIbBw4UJ8+OGHeOGFF9CuXbtq35OXlwegsoHq3r07fvzxR1VDtXPnTsTFxaFDhw5Kmt27d6v2s3PnTnTv3h0AY7AhOXPmDI4cOYKUlBSkp6fDarWq6vWXX37B8ePHlbpn/JBs8+bNSEpKQr9+/apMxzaIqtK2bVukpKSo6rKsrAz79+9X6rJHjx4oKSnBgQMHlDS7d++Gx+NBt27dAHhjZs+ePXA6nUqanTt34oILLlDmXawurkLJC9VP8oDUb7/9hueffx7Nmzev9j15eXkwmUzKIy6MIfJ14sQJnD59Wum72A5RqF5//XV06dIFnTp1qjZtg2mHamX6dKr33nvvPZGeni42b94sDh48KP7xj3+IXr16qVYzooZn3rx5omfPnuKzzz5TLSVaXl4uhBDit99+EytWrBDffPONOHLkiNi2bZu48sorxejRo5V9yMuxjxs3TuTl5YmPP/5YXHrppZrLsT/88MPi4MGD4qWXXtJcjp0xGH2WLl0qPvvsM3HkyBHx5ZdfirFjx4qsrCxRVFQkhBBi7ty5ol+/fmLXrl3im2++ESNHjhQjR45U3s/4ISG8K93169dPPPLII6rtbINIS1lZmfjuu+/Ed999J9LS0sS6devEd999p6yMtmrVKtGrVy+xbds28f3334vJkycHLF09fvx4MXjwYLF//36xZ88ecc0116iWYi8pKRHZ2dni3nvvFT/++KN47733REZGRsAy2hdddJFYu3atOHjwoHjqqac0l9GuLi9U96qKIYfDIW6//XZx2WWXiby8PNX1kbyC1VdffSXWrVsn8vLyxOHDh8Vbb70lLr30UnHfffcpx2AMNWxVxVBZWZlYunSp2Lt3rzhy5IjYuXOnGDJkiLjmmmtUq6CxHWrcquvLhBCitLRUZGRkiA0bNgS8vyG3QxyUasRefPFF0a9fP9GlSxcxfPhwsW/fvkhniWpZWlqa5r9NmzYJIYQ4fvy4GD16tLjkkktEenq6uPrqq8XDDz8sSktLVfs5evSo+Pvf/y66desmsrKyxNKlS4XT6VSl2b17t/jrX/8qunTpIq688krlGL4Yg9FnxowZok+fPqJLly6ib9++YsaMGeK3335TXv/jjz/E/PnzxcUXXywyMjLEHXfcIX7//XfVPhg/9Mknn4i0tDTxyy+/qLazDSItu3fv1uy7Zs2aJYTwLl/9xBNPiOzsbJGeni7+9re/BcTWqVOnxN133y26d+8uMjMzxezZs0VZWZkqTV5enhg1apRIT08Xffv2FatWrQrIy5YtW8Q111wjunTpInJzc8X27dtVr4eSF6p7VcXQkSNHgl4f7d69WwghxIEDB8SIESNEz549RdeuXcV1110nVq5cqRpwEIIx1JBVFUPl5eVi3Lhx4tJLLxVdunQRV1xxhZgzZ07Alxxshxq36voyIYTYuHGj6NatmygpKQl4f0NuhyQhhKide7CIiIiIiIiIiIi0cU4pIiIiIiIiIiKqcxyUIiIiIiIiIiKiOsdBKSIiIiIiIiIiqnMclCIiIiIiIiIiojrHQSkiIiIiIiIiIqpzHJQiIiIiIiIiIqI6x0EpIiIiIiIiIiKqcxyUIiIiIiIiIiKiOsdBKSIiImowZs+ejf79+0c6GwG2bNmCSy65BGfOnFG29e/fH7Nnz45grqLL0aNH0bFjR2zevDms9914441YtmxZLeWKiIiIjLBEOgNEREREVenYsWNI6davX1/LOdHH7Xbjn//8J8aMGYOmTZvW6rH27NmDlStX4ocffsDp06eRlJSETp06ITc3FwMHDqzVY9dXEyZMwL333ovbbrsNKSkpkc4OERER+eCgFBEREdVr/ne5vPXWW/j0008Dtrdv3x4PPfQQhBB1mb1qffTRRzh06BBGjhyp2v7+++9DkqQaO87WrVtx1113oXPnzrj11luRkJCAo0eP4osvvsCrr77aaAelrrzySsTFxWHDhg2YPn16pLNDREREPjgoRURERPXaX//6V9Xv+/fvx6effhqwvb7atGkTMjMzkZqaqtpus9lq9DgrVqxAhw4d8MorrwTsu6ioqEaPFU1MJhOuvfZavPXWW5g2bVqNDgQSERGRMZxTioiIiBoM/zml5HmI1q5di5dffhlXXnklMjIyMG7cOPzvf/+DEAJPP/00LrvsMnTr1g2TJ0/G6dOnA/a7Y8cO3HzzzejevTt69OiBiRMn4qeffqo2PxUVFfjkk0+QnZ0d8Jr/nFKbN29Gx44d8eWXX2LJkiW49NJL0b17d9xxxx04efJktcc6fPgwunbtqjnYlZSUpPrd4/Hg+eefR25uLrp27Yrs7GzMnTsXxcXFmmUfM2YMevTogczMTAwbNgzvvPOOKs3WrVsxdOhQdOvWDVlZWbjnnnuQn5+vSjN79mz06NED+fn5mDJlCnr06IFLL70UDz/8MNxutyptSUkJZs+ejZ49e6JXr16YNWsWSktLA/JWUFCA+++/H5dddhnS09ORk5ODyZMn4+jRo6p02dnZOHbsGPLy8qr+IxIREVGd4qAUERERNXjvvPMONmzYgFtuuQW33XYbPv/8c8yYMQNPPPEEPvnkE0yYMAE33ngjPvroIzz88MOq97755puYNGkSYmNjcc8992DKlCk4ePAgbr755oDBD38HDhyA0+nERRddFHJeFy1ahO+//x5Tp07FqFGj8NFHH2HhwoXVvq9169bYtWsXTpw4UW3auXPn4pFHHkFmZiYefPBBDB06FO+88w7Gjx8Pp9OppNu8eTMmTZqE4uJiTJo0CTNnzkTnzp3xySefqNLMmDEDJpMJd999N2688UZ8+OGHGDVqFEpKSlTHdbvdGD9+PBITE3HffffhkksuwXPPPYdXXnlFSSOEwJQpU/DWW29h0KBBmDFjBk6cOIFZs2YFlOPOO+/Ehx9+iKFDh2LevHm45ZZbcObMGfzvf/9TpUtPTwcAfPXVV9X+bYiIiKju8PE9IiIiavDy8/PxwQcfID4+HoD3TqFVq1bhjz/+wKZNm2CxeC+JTp06hXfeeQcLFiyAzWbDmTNn8P/+3//DiBEj8NBDDyn7GzJkCAYMGIBVq1aptvv75ZdfAABt27YNOa+JiYl47rnnlMfMPB4PXnzxRZSWlir51zJhwgQ8+OCDuOqqq5CZmYmePXuiT58+yMzMhMlU+T3knj178Nprr2H58uWqeaaysrLw97//He+//z4GDhyI0tJSLFq0CN26dcOLL74Iu92upJXn7XI6nVi+fDnS0tLw8ssvK2l69uyJSZMm4fnnn8e0adOU91VUVOC6667DHXfcAQAYNWoUhgwZgtdffx0333wzAOA///kPvvjiC9x77734+9//rqS79dZbVeUtKSnB3r17cd9992H8+PHK9kmTJgX8bVJTU2G1WnHw4MGq/vRERERUx3inFBERETV4AwYMUA3odOvWDQAwaNAgZUBK3u50OpVHz3bu3ImSkhLk5ubi5MmTyj+TyYSMjAx89tlnVR5XfhQwISEh5LzeeOONqnmPevXqBbfbjWPHjlX5vuHDh+PZZ59FVlYWvvrqKzzzzDMYPXo0rrnmGtUdQu+//z7i4+PRp08fVZm6dOmC2NhYpUyffvopzpw5g4kTJ6oGpAAo+Ttw4ACKioowatQoVZp+/frhwgsvxPbt2wPyOWrUKNXvPXv2VN1x9vHHH8NisajSmc1mjBkzRvW+mJgYWK1WfP7555qPHfpLSEjAqVOnqk1HREREdYd3ShEREVGDd95556l+lweogm0vLi5Gu3bt8OuvvwIA/va3v2nuNy4uLqTjh7MiYOvWrVW/N2vWDAACHoXT0rdvX/Tt2xfl5eX49ttvsWXLFmzcuBG33347tm7diqSkJPz2228oLS1F7969NfchT4p++PBhAMBf/vKXoMc7fvw4AOCCCy4IeO3CCy/El19+qdpmt9vRokUL1baEhATVoNKxY8eQkpKCpk2bqtL5H8Nms+Gee+7Bww8/jD59+iAjIwP9+vXD4MGDkZKSEpAfIQQnOSciIqpnOChFREREDZ7ZbNbc7vtYmy95EEn+f9myZZoDHcH2K0tMTATgHeRq1apVSHmtLk+haNKkCXr16oVevXqhefPmWLFiBT7++GMMGTIEHo8HSUlJWL58ueZ7/QeNalJ1f69wjR07Fv3798e2bdvw3//+F08++SRWr16NF154IWAer5KSEjRv3rxGj09ERETGcFCKiIiIKIh27doB8K5ep7WCXnUuvPBCAJWrAEaCPMl3QUEBAOD888/Hrl27kJmZiZiYmKDvO//88wEAP/30E/70pz9pppHv6jp06FDAnVeHDh0KuOsrFG3atMHu3btx5swZ1d1Shw4dCprPcePGYdy4cfj1118xePBgPPfcc6pBt/z8fDidTrRv3z7s/BAREVHt4ZxSREREREH07dsXcXFxWLVqlWpVOtnJkyerfH96ejqsVisOHDhQW1lU7Nq1S3P7jh07AFQ+/nbdddfB7XbjmWeeCUjrcrmUxwRzcnLQtGlTrFq1ChUVFap08l1b6enpSEpKwsaNG+FwOFTH/Pnnn9GvX7+wy3HZZZfB5XLh//7v/5RtbrcbL730kipdeXl5QL7OP/98NG3aVJUXAMrfv0ePHmHnh4iIiGoP75QiIiIiCiIuLg7z58/Hfffdh6FDh+L6669HixYtcPz4cezYsQOZmZmYO3du0Pfb7Xbk5ORg165dmD59eq3mdcqUKWjbti2uuOIKtGvXDuXl5di5cyc++ugjdO3aFVdccQUA4JJLLsHIkSOxatUq5OXloU+fPrBarfj111/x/vvv48EHH8SAAQMQFxeH+++/H3PmzMHw4cNxww03oFmzZvj+++/xxx9/4OGHH4bVasU999yD+++/H2PGjEFubi6Kioqwfv16tGnTBmPHjg27HP3790dmZiYeffRRHDt2DB06dMAHH3yA0tJSVbpff/0VY8eOxYABA9ChQweYzWZs27YNhYWFyM3NVaXduXMnWrduHfBIHxEREUUWB6WIiIiIqjBw4EC0bNkSq1evxtq1a+FwOJCamopevXph6NCh1b5/2LBhuPPOO/G///0vYGL1mrRo0SL85z//wdatW/H7779DCIF27drh9ttvx4QJE1SrDC5cuBDp6enYuHEjHn/8cZjNZrRp0waDBg1CZmamkm7EiBFISkrC6tWr8cwzz8BiseDCCy9UDTYNHToUMTExWLNmDZYvX47Y2FhcddVVuPfee5VJ2sNhMpnwr3/9C4sXL8bbb78NSZLQv39/zJ49G4MHD1bStWrVCrm5udi1axfefvttmM1mXHjhhXjiiSdw7bXXKuk8Hg/+/e9/Y/jw4ZzonIiIqJ6RRDizZhIRERFRWNxuN66//npcd911mDFjRqSz0+hs27YNM2fOxIcffoiWLVtGOjtERETkg3NKEREREdUis9mM6dOnY8OGDThz5kyks9PorFmzBqNHj+aAFBERUT3EO6WIiIiIiIiIiKjO8U4pIiIiIiIiIiKqcxyUIiIiIiIiIiKiOsdBKSIiIiIiIiIiqnMclCIiIiIiIiIiojrHQSkiIiIiIiIiIqpzHJQiIiIiIiIiIqI6x0EpIiIiIiIiIiKqcxyUIiIiIiIiIiKiOsdBKSIiIiIiIiIiqnMclCIiIiIiIiIiojrHQSkiIiIiIiIiIqpz/x8kQP/wb7Al+AAAAABJRU5ErkJggg==\n"
          },
          "metadata": {}
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "The red histogram represents fraudulent transactions. The x-axis shows the time in seconds, and the y-axis represents the number of transactions. Fraudulent transactions are relatively infrequent compared to normal transactions, as seen by the lower y-axis values. There are peaks at certain times, suggesting that fraudulent activity may occur in bursts or at specific moments rather than evenly distributed throughout the time frame covered.\n",
        "\n",
        "The blue histogram represents normal transactions which have a higher volume. The pattern here is more consistent, with what appears to be a cyclical pattern suggesting higher transaction volumes at regular intervals. This could correspond to peak transaction times during the day, such as morning and evening hours when people are more likely to use their credit cards."
      ],
      "metadata": {
        "id": "v7l_BJrChuIr"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "plt.figure(figsize=(10, 6))\n",
        "plt.boxplot([data[data[\"Class\"]==1][\"Amount\"], data[data[\"Class\"]==0][\"Amount\"]],\n",
        "            labels=['Fraud', 'Normal'])\n",
        "\n",
        "plt.title('Transaction Amounts: Fraud vs Normal')\n",
        "plt.ylabel('Amount')\n",
        "plt.yscale('log')\n",
        "plt.show()"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:40:15.980245Z",
          "iopub.execute_input": "2023-11-24T17:40:15.980696Z",
          "iopub.status.idle": "2023-11-24T17:40:16.867461Z",
          "shell.execute_reply.started": "2023-11-24T17:40:15.980661Z",
          "shell.execute_reply": "2023-11-24T17:40:16.866323Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 545
        },
        "id": "taluIhM4huIs",
        "outputId": "f0a44e15-b9a7-4ee8-ccd1-4a7e595498f5"
      },
      "execution_count": 7,
      "outputs": [
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 1000x600 with 1 Axes>"
            ],
            "image/png": "iVBORw0KGgoAAAANSUhEUgAAA1EAAAIQCAYAAABg9uTXAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjcuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/bCgiHAAAACXBIWXMAAA9hAAAPYQGoP6dpAABM/klEQVR4nO3deXxM9/7H8XcyEVuCELS4SmkSJRE7qTQEpUERtLf3Bm1TVVV6K3629rbVe69oL9VaerW0WltdS+IWsdSSCo1da2kstbWWWmJPQmRmfn94ZJgmyGFkhnk9Hw8Pc77na76fGTPJvOd7zvd4WK1WqwAAAAAABeLp7AIAAAAA4H5CiAIAAAAAAwhRAAAAAGAAIQoAAAAADCBEAQAAAIABhCgAAAAAMIAQBQAAAAAGEKIAAAAAwABCFAAAAAAYQIgCgPvA0KFDFRkZ6ewy4EbGjx+vwMBAZ5fxQDty5IgCAwOVkJDg7FIAGOTl7AIAIFdBP7BNmzZNTZo0ucfVFL4TJ05ozpw5at26tWrVquXscvLYv3+/oqKi5O3trXXr1qlUqVLOLsmhtm7dqnXr1qlXr1739LEdOXJErVq1yndf3bp1NWfOnHs29v2oR48e2rhxo1q2bKlJkybZ7ct9LgcPHqzY2FgnVQjAHRGiALiMDz/80G77f//7n9atW5envUaNGoVZVqE5efKkJkyYoMqVK+cJUf/4xz9ktVqdVNk13377rcqXL6/z589r2bJl6t69u1PrcbRt27ZpwoQJ6tKlS6EExA4dOujJJ5+0aytbtuw9H/d+tXr1au3cuVN16tRxdikAQIgC4Do6depkt/3TTz9p3bp1edr/KCsrS8WLF7+XpTldkSJFnDq+1WrVwoUL1aFDBx05ckTffvvtAxeiCtvjjz9+29d2LovFoqtXr6po0aL3uCrXVKlSJWVkZGjChAl5ZqMc6cqVKypSpIg8PTnbAcCt8VMCwH2lR48e6tChg3bu3Km//vWvqlu3rj766CNJ0ooVK/TKK6+oefPmqlOnjlq3bq2JEyfKbDbnex+//PKLevToobp16yo8PFyTJ0/OM9706dPVvn171a1bV40aNVJ0dLQWLlxo23/06FG99957atu2rUJCQtSkSRMNGDBAR44cyXNfFy5c0MiRIxUZGak6deroySef1ODBg3XmzBlt2LBB3bp1kyQNGzZMgYGBdudK5HdOVGZmpkaNGqWIiAjVqVNHbdu21RdffJFnxiowMFDvv/++VqxYoQ4dOqhOnTpq37691qxZU+DnfcuWLTp69KiioqIUFRWlzZs36/fff8/TLzIyUn369NGGDRsUHR2tkJAQdezYURs2bJAkLV++XB07dlRwcLCio6P1888/57mP1NRU/eUvf1FoaKgaNmyovn37av/+/XZ9bnaOWH7n8RTk8Y8fP94249mqVSvb85/7/7hu3To9//zzatiwoerVq6e2bdvaXne5jh07lqfOO5Vb87fffqv27dsrODhYKSkpkqQvvvhCf/7zn9WkSROFhIQoOjpaS5cutfv3tzrXJjAwUOPHj7dr27x5s7p27arg4GC1bt1as2fPLlCd77//vurVq6esrKw8+wYOHKgnnnjC9v7bsWOHYmNjbXVHRkZq2LBhBRqnZMmS6tWrl1avXq1du3bdtv9vv/2mAQMGqHHjxqpbt66effZZJScn2/XZsGGDAgMDtXjxYo0dO1bh4eGqW7euLl26pKFDh6pevXo6duyY+vTpo3r16ik8PFwzZ86UJO3Zs0c9e/ZUaGioWrZsafczQZLOnTunDz74QB07dlS9evVUv359vfzyy9q9e3eBHi8A18dMFID7zrlz59S7d2+1b99ezzzzjMqVKydJSkxMVIkSJfTiiy+qRIkSWr9+vcaNG6dLly5pyJAhdvdx/vx5vfzyy2rTpo2efvppLVu2TKNHj1ZAQIAiIiIkSXPmzNE///lPtW3bVj179tSVK1e0Z88e/fTTT+rYsaOkax8Mt23bpvbt2+uhhx7S0aNH9c0336hnz55avHixbYYsIyNDf/3rX7V//3517dpVjz/+uM6ePatVq1bpxIkTqlGjhgYMGKBx48bpueeeU4MGDSRJ9evXz/c5sFqt6tu3ry181apVSykpKfrwww914sQJDR8+3K7/li1btHz5cv3lL39RyZIlNX36dA0YMECrV6+Wn5/fbZ/zhQsXqmrVqgoJCVFAQICKFSumRYsW6eWXX87T9/Dhw4qLi9Of//xnPfPMM/ryyy/16quvasSIERo7dqyef/55SdLnn3+uv/3tb1q6dKntm/8ffvhBvXv3VpUqVfT666/r8uXLmjFjhp5//nklJCSoSpUqt601P7d7/G3atNGhQ4e0aNEiDRs2zPaclC1bVvv27VOfPn0UGBioAQMGyNvbW4cPH9bWrVvtxhgyZIg2btyoPXv2FKimrKwsnTlzxq7N19fXNuu4fv16LVmyRH/961/l5+enypUrS7p2TmBkZKQ6duyoq1evavHixXrjjTf02WefqUWLFoafmz179ig2NlZly5ZV//79lZOTo/Hjx9veV7cSFRWlmTNnKjk5WU8//bTdY1u9erW6dOkik8mk9PR0xcbGys/PT6+88opKlSqlI0eO6Lvvvitwnb169dLXX3+t8ePH33I26vTp0/rzn/+srKws9ejRQ35+fkpMTFTfvn01btw4tWnTxq7/p59+qiJFiig2NlbZ2dm2599sNqt3795q2LChBg0apIULF+r9999X8eLFNXbsWHXs2FFPPfWUZs+erSFDhig0NFR/+tOfJF0LcStWrFC7du1UpUoVnT59Wv/9738VExOjxYsXq2LFigV+3ABclBUAXNSIESOsAQEBdm0xMTHWgIAA6zfffJOnf1ZWVp62v//979a6detar1y5kuc+EhMTbW1XrlyxPvHEE9b+/fvb2vr27Wtt3779LWvMb8xt27bluf9PPvnEGhAQYF2+fHme/haLxWq1Wq3bt2+3BgQEWOfPn5+nz5AhQ6wtW7a0bX/33XfWgIAA66effmrXr3///tbAwEDr4cOHbW0BAQHW2rVr27WlpaVZAwICrNOnT7/l47Nardbs7Gxr48aNrR999JGtbeDAgdZnnnkmT9+WLVtaAwICrFu3brW1paSkWAMCAqwhISHWo0eP2tpnz55tDQgIsK5fv97W1qlTJ2uzZs2sZ8+etas1KCjIOnjw4Js+H7nGjRuX5zVT0Mc/ZcoUa0BAgPW3336z+/dTp061BgQEWNPT0/N9fnLlvq5u57fffrMGBATk+yf3uQgICLAGBQVZ9+3bl+ff//E1l52dbe3QoYO1Z8+eecbI77UUEBBgHTdunG37tddeswYHB9v93/zyyy/WWrVq3fbxWCwWa3h4uN37xmq1WpOSkqwBAQHWTZs2Wa3W66/X7du33/L+8hMTE2N7H44fP94aEBBg3blzp93jnDJliq3/v/71L7uxrVar9dKlS9bIyEhry5YtrWaz2Wq1Wq3r16+3BgQEWFu1apXnOR0yZIg1ICDAOmnSJFvb+fPnrSEhIdbAwEDr4sWLbe379+/P85xeuXLFNk6u3377zVqnTh3rhAkT7Npu9v8EwLVxOB+A+463t7eio6PztBcrVsx2+9KlSzpz5owaNmyorKwsHThwwK5viRIl7M5H8fb2VnBwsH777TdbW6lSpfT7779r+/btN63lxjGvXr2qs2fPqmrVqipVqpTdoWrLly9XUFBQnm/BJcnDw+M2jzivNWvWyGQyqUePHnbtL730kqxWa55D9cLCwlS1alXbdlBQkHx8fOwe763GOnfunDp06GBr69Chg3bv3q19+/bl6V+zZk3Vq1fPtl23bl1JUtOmTVWpUqU87bk1nDx5UmlpaerSpYvKlCljV2tYWJi+//7729Z6M3fz+HMXmVi5cqUsFstN+02fPr3As1CS9Nxzz2nq1Kl2f4KCgmz7GzVqpJo1a+b5dze+5s6fP6+LFy+qQYMG+R4aeTtms1lr165V69at7f5vatSooebNm9/233t4eKhdu3b6/vvvlZGRYWtfsmSJKlasaJtR9fX1lSQlJyfr6tWrhuvM1atXL5UuXVoTJky4aZ/vv/9eISEhatiwoa2tZMmSeu6553T06FH98ssvdv07d+5s95ze6Mbz/kqVKqXq1aurePHidrNujz76qEqVKmX3WvL29rbNrprNZp09e1YlSpRQ9erV7+j/CYDr4XA+APedihUrytvbO0/7vn379PHHH2v9+vW6dOmS3b6LFy/abT/00EN5wkvp0qXtPgT37t1bP/zwg7p3765HHnlETzzxhDp06GD7YChJly9f1meffaaEhASdOHHC7nykG8f89ddf9dRTT93ZA87H0aNHVaFCBfn4+Ni1565cePToUbv2hx9+OM99lC5dWhcuXLjtWN9++62qVKliO4xNkqpWrarixYtr4cKFGjhw4C3Hyv0A/dBDD9m159aeW8OxY8ckSdWrV89TQ40aNbR27VplZmaqRIkSt635j+7m8UdFRWnu3Ll6++23NWbMGDVr1kxt2rRRu3bt7moBgkceeURhYWE33X+zQxdXr16t//znP0pLS1N2drat/U7C+JkzZ3T58mU98sgjefZVr169QME1KipKX3/9tVatWqWOHTsqIyND33//vZ577jlbTY0bN1bbtm01YcIEffXVV2rcuLFat26tjh075vtevhlfX1/17NlT48eP188//5zvKorHjh2zBfQbPfroo7b9AQEBtvabPc9FixbNs1qir69vvj87fH197V5L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\n"
          },
          "metadata": {}
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "The median fraudulent transaction is of **9.25** with the largest amount in a fraudulent transaction being **2125.87**.\n",
        "The median amount for a normal transaction is **22** whereas the largest amount is **25691** which is substantially higher than the maximum for a fraudulent transaction. This could be due to the larger sample size or it could indicate that fraudsters avoid extremely large transactions that might trigger security mechanisms."
      ],
      "metadata": {
        "id": "qCfXVU4mhuIt"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "\n",
        "pd.to_numeric(data.columns, errors=\"ignore\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "hcNR71jxoyIi",
        "outputId": "1d9e8d20-ccab-4645-d4f4-5edf2c881794"
      },
      "execution_count": 8,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "Index(['Time', 'V1', 'V2', 'V3', 'V4', 'V5', 'V6', 'V7', 'V8', 'V9', 'V10',\n",
              "       'V11', 'V12', 'V13', 'V14', 'V15', 'V16', 'V17', 'V18', 'V19', 'V20',\n",
              "       'V21', 'V22', 'V23', 'V24', 'V25', 'V26', 'V27', 'V28', 'Amount',\n",
              "       'Class'],\n",
              "      dtype='object')"
            ]
          },
          "metadata": {},
          "execution_count": 8
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Data Preparation\n",
        "- Split data into training, validation, and test sets (60/20/20).\n",
        "- Use stratified sampling to maintain the class distribution in each split.\n",
        "- Scale the training, validation, and test sets to a mean of zero mean and unit variance."
      ],
      "metadata": {
        "id": "3Gk5J0H4huIt"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# split data into train, validation, and test\n",
        "X_temp, X_test, y_temp, y_test = train_test_split(X, y, test_size=0.4, random_state=42)\n",
        "X_train, X_val, y_train, y_val = train_test_split(X_temp, y_temp, test_size=0.25, random_state=42)\n",
        "\n",
        "# normalize data\n",
        "scaler = StandardScaler()\n",
        "X_train = scaler.fit_transform(X_train)\n",
        "X_val = scaler.transform(X_val)\n",
        "X_test = scaler.transform(X_test)"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:40:26.936041Z",
          "iopub.execute_input": "2023-11-24T17:40:26.936480Z",
          "iopub.status.idle": "2023-11-24T17:40:27.382414Z",
          "shell.execute_reply.started": "2023-11-24T17:40:26.936444Z",
          "shell.execute_reply": "2023-11-24T17:40:27.380973Z"
        },
        "trusted": true,
        "id": "DtwAB5XxhuIu"
      },
      "execution_count": 9,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Model Architecture"
      ],
      "metadata": {
        "id": "NCv6s44FhuIv"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "model = tf.keras.Sequential(\n",
        "    [\n",
        "        # Adjusted number of neurons\n",
        "        tf.keras.layers.Dense(128, activation=\"relu\", input_shape=(X.shape[-1],),\n",
        "                              kernel_regularizer=regularizers.l2(0.001)),  # L2 regularization\n",
        "        tf.keras.layers.Dropout(0.2),  # Adjusted dropout rate\n",
        "        tf.keras.layers.Dense(64, activation=\"relu\",\n",
        "                              kernel_regularizer=regularizers.l2(0.001)),  # L2 regularization\n",
        "        tf.keras.layers.Dropout(0.2),  # Adjusted dropout rate\n",
        "        tf.keras.layers.Dense(1, activation=\"sigmoid\"),\n",
        "    ]\n",
        ")\n",
        "model.summary()"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:40:31.559911Z",
          "iopub.execute_input": "2023-11-24T17:40:31.561232Z",
          "iopub.status.idle": "2023-11-24T17:40:31.853722Z",
          "shell.execute_reply.started": "2023-11-24T17:40:31.561182Z",
          "shell.execute_reply": "2023-11-24T17:40:31.852765Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "f7dqm-l1huIw",
        "outputId": "cff21731-8913-437c-b3ae-8b73c24b5ee2"
      },
      "execution_count": 10,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Model: \"sequential\"\n",
            "_________________________________________________________________\n",
            " Layer (type)                Output Shape              Param #   \n",
            "=================================================================\n",
            " dense (Dense)               (None, 128)               3968      \n",
            "                                                                 \n",
            " dropout (Dropout)           (None, 128)               0         \n",
            "                                                                 \n",
            " dense_1 (Dense)             (None, 64)                8256      \n",
            "                                                                 \n",
            " dropout_1 (Dropout)         (None, 64)                0         \n",
            "                                                                 \n",
            " dense_2 (Dense)             (None, 1)                 65        \n",
            "                                                                 \n",
            "=================================================================\n",
            "Total params: 12289 (48.00 KB)\n",
            "Trainable params: 12289 (48.00 KB)\n",
            "Non-trainable params: 0 (0.00 Byte)\n",
            "_________________________________________________________________\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "![download.png](attachment:0c11b26a-ffb5-4c6e-9390-329758995245.png)1. Input Layer: A dense layer with 128 neurons, ReLU activation function, and L2 regularization with a strength of 0.001. It takes input data with a shape determined by the number of features in the dataset.\n",
        "2. Dropout Layer: A dropout layer with a dropout rate of 0.2, which helps prevent overfitting by randomly deactivating 20% of neurons during training.\n",
        "3. Hidden Layer: Another dense layer with 64 neurons, ReLU activation, and L2 regularization with a strength of 0.001.\n",
        "4. Another Dropout Layer: Similar to the previous dropout layer, with a rate of 0.2.\n",
        "5. Output Layer: A dense layer with a single neuron and sigmoid activation function, used for binary classification.\n",
        "\n",
        "### ReLU activation function\n",
        "- Decides whether the neuron should be activated or not.\n",
        "- Introduces non-linear transformation making it capable to learn and perform more complex tasks.\n",
        "- Maps any number to zero if it is negative, and otherwise maps it to itself.\n",
        "![image.png](attachment:cb1f20be-375c-4c90-a142-84521cd8fa33.png)\n",
        "\n",
        "### Regularization\n",
        "These are the most common regularization techniques.\n",
        "![image.png](attachment:d8711229-bbe5-46f4-87ad-c856fa8333ba.png)\n",
        "- During the model definition I introduced parameter regularization (weight decay) and neural network regularization (dropout).\n",
        "- During the model training I added early stopping.\n",
        "\n",
        "#### Weight decay\n",
        "\n",
        "Weight decay is a regularization term implemented from tensorflow.keras with the following formula:\n",
        "\n",
        "![image.png](attachment:fb3631a6-ab49-47e0-8280-eb5a62e4c249.png)\n",
        "`math_ops.reduce_sum(math_ops.square(x))` calculates the sum of the squares of all elements in the tensor x\n",
        "\n",
        "#### Dropout regularization\n",
        "- Deep neural nets with a large number of parameters are powerful but prone to overfitting.\n",
        "- Dropout is a technique for addressing this problem. The key idea is to randomly drop units (along with their connections) from the neural network during training. This prevents units from co-adapting too much.\n",
        "- During training, dropout samples form an exponential number of different “thinned” networks. At test time, instead of using all these thinned networks, we can approximate their effect by using a single unthinned network with smaller weights."
      ],
      "metadata": {
        "id": "5iwr9Ov6huIx"
      }
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Metrics"
      ],
      "metadata": {
        "id": "2x_Bq4PqhuIz"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "metrics = [\n",
        "    tf.keras.metrics.Accuracy(),\n",
        "]"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T17:57:20.067711Z",
          "iopub.execute_input": "2023-11-24T17:57:20.068096Z",
          "iopub.status.idle": "2023-11-24T17:57:20.097527Z",
          "shell.execute_reply.started": "2023-11-24T17:57:20.068067Z",
          "shell.execute_reply": "2023-11-24T17:57:20.096433Z"
        },
        "trusted": true,
        "id": "euSjVEYPhuIz"
      },
      "execution_count": 11,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "In this imbalanced dataset using accuracy as the evaluation metric is not appropriate because it can be misleading. A model that predicts all instances as negative 0 would still achieve a high accuracy of 99%, even though it's not providing meaningful results. This is why I am prioritizing the following metrics:\n",
        "\n",
        "![image.png](attachment:3b1dc5fb-9d61-48f3-9666-63a12a9b4cd7.png)\n",
        "\n",
        "- `Precision` calculates the ratio of true positives to the total number of positive predictions (true positives + false positives).\n",
        "\n",
        "- `Recall` calculates the ratio of true positives to the total number of actual positive instances (true positives + false negatives)."
      ],
      "metadata": {
        "id": "AToyPYO6huI0"
      }
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Model training\n",
        "- Use the Adam optimizer with a learning rate of 0.0001.\n",
        "- Utilize binary cross-entropy as the loss function.\n",
        "- Set up an early stopping callback that monitors the validation loss for minimization.\n",
        "- Specify a patience of 5 epochs before stopping training if the validation loss does not improve.\n",
        "- Set class weights such that class 0 has a weight of 1, and class 1 has a weight of 5, giving more importance to the minority class.\n",
        "- Train for a maximum of 100 epochs."
      ],
      "metadata": {
        "id": "0vOdrBMchuI0"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# compile the model\n",
        "model.compile(optimizer=tf.keras.optimizers.Adam(learning_rate=0.001), loss='CategoricalCrossentropy', metrics = metrics)\n",
        "\n",
        "# configure early stopping\n",
        "es = tf.keras.callbacks.EarlyStopping(monitor='val_loss', mode='min', verbose=1, patience=5)\n",
        "\n",
        "# # calculate class weights\n",
        "# neg, pos = np.bincount(y_train)\n",
        "# total = neg + pos\n",
        "# class_weight = {0: 1, 1: 5}\n",
        "\n",
        "# train the model\n",
        "history = model.fit(X_train, y_train, validation_data=(X_val, y_val), epochs=100, callbacks=[es])"
      ],
      "metadata": {
        "_kg_hide-output": true,
        "execution": {
          "iopub.status.busy": "2023-11-24T17:57:26.618088Z",
          "iopub.execute_input": "2023-11-24T17:57:26.618689Z",
          "iopub.status.idle": "2023-11-24T17:58:55.650404Z",
          "shell.execute_reply.started": "2023-11-24T17:57:26.618645Z",
          "shell.execute_reply": "2023-11-24T17:58:55.649256Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "Y_KAIzXthuI1",
        "outputId": "5cedea4f-3417-48e2-8306-8bc5e3cf8333"
      },
      "execution_count": 12,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Epoch 1/100\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/usr/local/lib/python3.10/dist-packages/tensorflow/python/util/dispatch.py:1260: SyntaxWarning: In loss categorical_crossentropy, expected y_pred.shape to be (batch_size, num_classes) with num_classes > 1. Received: y_pred.shape=(None, 1). Consider using 'binary_crossentropy' if you only have 2 classes.\n",
            "  return dispatch_target(*args, **kwargs)\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "4006/4006 [==============================] - 15s 4ms/step - loss: 70.3488 - accuracy: 0.9667 - val_loss: 193.2969 - val_accuracy: 0.9982\n",
            "Epoch 2/100\n",
            "4006/4006 [==============================] - 10s 3ms/step - loss: 386.3694 - accuracy: 0.9983 - val_loss: 608.9213 - val_accuracy: 0.9982\n",
            "Epoch 3/100\n",
            "4006/4006 [==============================] - 8s 2ms/step - loss: 887.0489 - accuracy: 0.9983 - val_loss: 1192.9917 - val_accuracy: 0.9982\n",
            "Epoch 4/100\n",
            "4006/4006 [==============================] - 7s 2ms/step - loss: 1551.9850 - accuracy: 0.9983 - val_loss: 1937.7487 - val_accuracy: 0.9982\n",
            "Epoch 5/100\n",
            "4006/4006 [==============================] - 7s 2ms/step - loss: 2375.5964 - accuracy: 0.9983 - val_loss: 2839.0659 - val_accuracy: 0.9982\n",
            "Epoch 6/100\n",
            "4006/4006 [==============================] - 7s 2ms/step - loss: 3355.1165 - accuracy: 0.9983 - val_loss: 3897.1538 - val_accuracy: 0.9982\n",
            "Epoch 6: early stopping\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "print(history.history['accuracy'])\n",
        "print(history.history['loss'])"
      ],
      "metadata": {
        "execution": {
          "iopub.status.busy": "2023-11-24T18:01:03.183743Z",
          "iopub.execute_input": "2023-11-24T18:01:03.184181Z"
        },
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "38HVEXj_huI1",
        "outputId": "90552a49-bc7f-4947-d288-2dc4562bca47"
      },
      "execution_count": 13,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "[0.9666518568992615, 0.9982678294181824, 0.9982678294181824, 0.9982678294181824, 0.9982678294181824, 0.9982678294181824]\n",
            "[70.34882354736328, 386.3694152832031, 887.0488891601562, 1551.9849853515625, 2375.596435546875, 3355.116455078125]\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "test_loss, test_acc = model.evaluate(X_test, y_test,  verbose=2)\n",
        "print('\\nTest accuracy:', test_acc)\n"
      ],
      "metadata": {
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "TXlJajvShuI2",
        "outputId": "ce4ebd85-3477-423c-ce1b-27320a1fbd92"
      },
      "execution_count": 14,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "3561/3561 - 3s - loss: 3897.0574 - accuracy: 0.9983 - 3s/epoch - 830us/step\n",
            "\n",
            "Test accuracy: 0.9983234405517578\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# Model Evaluation and Loss Visualization\n",
        "- Use the trained model to make predictions on the test dataset.\n",
        "- Calculate precision, recall, and F1 score as evaluation metrics for the model's performance on the test data.\n",
        "- Visualize the training and validation loss over the training epochs."
      ],
      "metadata": {
        "id": "NE02DfnqhuI3"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# predict test data\n",
        "y_pred = (model.predict(X_test))\n",
        "\n",
        "# score precision, recall, and f1\n",
        "precision = precision_score(y_test, y_pred)\n",
        "recall = recall_score(y_test, y_pred)\n",
        "f1 = f1_score(y_test, y_pred)\n",
        "\n",
        "print(f'Precision: {precision}')\n",
        "print(f'Recall: {recall}')\n",
        "print(f'F1 Score: {f1}')\n",
        "\n",
        "# Plot only the losses from history\n",
        "losses = history.history['loss']\n",
        "val_losses = history.history['val_loss']\n",
        "\n",
        "plt.figure(figsize=(10, 7))\n",
        "plt.plot(losses, label='Training Loss')\n",
        "plt.plot(val_losses, label='Validation Loss')\n",
        "plt.xlabel('Epochs')\n",
        "plt.ylabel('Loss')\n",
        "plt.legend()\n",
        "plt.grid(True)\n",
        "plt.show()"
      ],
      "metadata": {
        "trusted": true,
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 743
        },
        "id": "s8FRWP63huI3",
        "outputId": "9dd30533-c4f2-4264-d168-ec90eba90896"
      },
      "execution_count": 16,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "3561/3561 [==============================] - 8s 2ms/step\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/usr/local/lib/python3.10/dist-packages/sklearn/metrics/_classification.py:1344: UndefinedMetricWarning: Precision is ill-defined and being set to 0.0 due to no predicted samples. Use `zero_division` parameter to control this behavior.\n",
            "  _warn_prf(average, modifier, msg_start, len(result))\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Precision: 0.0\n",
            "Recall: 0.0\n",
            "F1 Score: 0.0\n"
          ]
        },
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 1000x700 with 1 Axes>"
            ],
            "image/png": "iVBORw0KGgoAAAANSUhEUgAAA1sAAAJaCAYAAADZF10UAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjcuMSwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/bCgiHAAAACXBIWXMAAA9hAAAPYQGoP6dpAACjJ0lEQVR4nOzdd1iV5QPG8S/nAAICLnBvVFBRwZmGWaaWmplajsyVmWaOhqmZW1MzLSsry8w9KjPNhqUN05ypKO49cQAOQGSdc35/vMkvmqLAe4D7c11el8/h5XAffURv3+c8j4vD4XAgIiIiIiIimcpidgAREREREZHcSGVLREREREQkC6hsiYiIiIiIZAGVLRERERERkSygsiUiIiIiIpIFVLZERERERESygMqWiIiIiIhIFlDZEhERERERyQKuZgfICex2O6mpqVgsFlxcXMyOIyIiIiIiJnE4HNjtdlxdXbFY/v3elcrWLUhNTSUiIsLsGCIiIiIi4iRq1KiBu7v7v16jsnULbjbWGjVqYLVaTU4DNpuNiIgIp8kjzk9zRjJC80UySnNGMkpzRjLKmebMzSz/dVcLVLZuyc2lg1ar1fTf3D9ytjzi/DRnJCM0XySjNGckozRnJKOcac7cytuLtEGGiIiIiIhIFlDZEhERERERyQIqWyIiIiIiIllA79nKJA6Hg9TUVGw2W5Z/rZtfIzEx0WnWrErWsVqtuLq66tgBERERkRzGacrWhx9+yPTp0+nevTuvvPIKAElJSUyZMoVvvvmG5ORkwsLCGDNmDH5+fmmfFxkZydixY9m6dSteXl488sgjvPjii7i6/v+lbd26lSlTpnDkyBFKlCjBM888Q/v27TMte3JyMufPnychISHTnvPfOBwOXF1dOXXqlP4Bnkd4eXlRokSJ/9xeVERERESch1OUrT179rBs2TICAwPTPT5p0iTWr1/PjBkz8PHxYcKECQwYMIBly5YBxh2evn374ufnx7Jly7h06RLDhg3Dzc2NF154AYAzZ87Qt29fOnfuzLRp09i8eTMjR47E39+fxo0b33F2u93OiRMnsFqtlCxZEnd39ywvQA6Hgxs3buDp6amylcs5HA6Sk5OJiorixIkTVK5c+Za2GRURERER85letq5fv85LL73ExIkTef/999Mej4uL4/PPP2fatGk0bNgQMMpXq1atCA8PJyQkhI0bN3L06FHmzp2Ln58fVatWZfDgwUybNo0BAwbg7u7OsmXLKF26NMOHDwcgICCAHTt2MG/evEwpW8nJydjtdsqUKYOXl9cdP9+tuHlqtYeHh8pWHuDp6YmbmxunTp0iOTkZDw8PsyOJiIiIyC0wvWyNHz+eJk2a0KhRo3Rla+/evaSkpNCoUaO0xwICAihZsmRa2QoPD6dKlSrplhWGhYUxduxYjh49SrVq1QgPD08ra3+8ZtKkSRnO+nfvx7LZbDgcDlxcXHA4HBl+zttx8+tk19cT892cXzab7bbeF3jzc7LjPYWS82m+SEZpzkhGac5IRjnTnMlIBlPL1tdff83+/ftZvnz5Xz4WHR2Nm5sbvr6+6R4vUqQIUVFRadf8sWgBaeP/uiY+Pp7ExMQM3SWIiIj428ddXV25ceMGdrv9lp8rM9y4cSNbv56YJykpiZSUFA4ePHhHz/NPc1jk72i+SEZpzkhGac5IRuW0OWNa2Tp//jyvvvoqH3/8Mfny5TMrRobUqFHjL7v/JSYmcurUKTw9PbNteZczv2eradOm9OjRgx49etzS9Vu3bqVHjx5s27btL8Va/s9iseDm5kalSpVua57ZbDYiIiL+dg6L/Jnmi2SU5oxklOaMZJQzzZmbWW6FaWVr3759xMTEpNsV0GazsX37dhYvXsycOXNISUkhNjY23T/CY2Ji8Pf3B4w7VHv27En3vNHR0QDprrn52B+v8fb2zvA/Wq1W619+c61WKy4uLmk/stOdfM0/b0byZwMGDGDgwIEZft7PP/88QyWwdu3abNy4EV9f3yz99du6dSvdu3dn+/btObLU3fy9/rs5mBF3+vmSt2i+SEZpzkhGac5IRuW0OWNa2brrrrtYvXp1usdefvllKlasSJ8+fShRogRubm5s3ryZBx54AIDjx48TGRlJSEgIACEhIcyaNYuYmBiKFCkCwKZNm/D29qZSpUpp1/zyyy/pvs6mTZvSniOv2rhxY9rPv/nmG95++23WrFmT9tgfN/u4+V6hP26n/08KFy6coRzu7u5pxVhEREREJDcxbQ9pb29vqlSpku6Hl5cXBQsWpEqVKvj4+NChQwemTJnCli1b2Lt3LyNGjCA0NDStKIWFhVGpUiWGDh3KwYMH2bBhAzNmzKBr165p5xF17tyZM2fOMHXqVI4dO8bixYv59ttv6dmzp1kv3Sn4+/un/fDx8cHFxSVtfPz4cWrXrs369etp3749NWrUYMeOHZw+fZpnnnmGRo0aERoaSocOHdi0aVO6523atCnz5s1LGwcGBvLZZ5/x7LPPUqtWLVq0aMEPP/yQ9vGtW7cSGBhIbGwsACtWrKBu3bps2LCBli1bEhoaSu/evbl06VLa56SmpjJx4kTq1q1LgwYNeP311xk2bBj9+/e/7V+Pa9euMXToUOrVq0etWrV46qmnOHnyZNrHz507R79+/ahXrx4hISG0bt2a9evXp33uiy++yF133UXNmjVp0aIFn3/++W1nEREREZHcwfTdCP/NiBEjsFgsDBo0KN2hxjdZrVZmzZrF2LFj6dSpE56enrRr145BgwalXVOmTBk++OADJk+ezIIFCyhevDgTJ07MlG3f/43D4eBGStbsluJwOEhItoFrarqld55u1kxdijd9+nSGDRtGmTJl8PX15cKFCzRp0oTnn38ed3d3Vq5cSb9+/VizZg0lS5b8x+eZOXMmL730EkOHDmXhwoUMGTKEn376iYIFC/7t9YmJiXz88cdMnToVi8XCSy+9xGuvvcb06dMBmD17NqtXr2by5MlUrFiRBQsWsG7dOho0aHDbr3X48OGcOnWK999/H29vb15//XWefvppvv76a9zc3Bg/fjwpKSksWrQILy8vjh49mnb376233uLYsWPMnj2bQoUKcfr0aRITE287i4iIiIjkDk5VthYuXJhunC9fPsaMGZOuYP1ZqVKlmD179r8+b4MGDVi5cmVmRLwlDoeDR2dtZsepK9n2NQHqlivEZ/0aZlrhGjRoEHfffXfauGDBggQFBaWNn3vuOdatW8ePP/7IE0888Y/P065dOx566CEAXnjhBRYuXMiePXu45557/vb6lJQUxo0bR9myZQHo2rUr7733XtrHFy1axNNPP03z5s0BGD169F+WimbEyZMn+fHHH1m6dCm1a9cGYNq0adx7772sW7eOli1bEhkZyQMPPJD2XrcyZcqkfX5kZCRVq1alRo0aAJQuXfq2s4iIiIhI7uFUZSs3ca49Am/PzfJw0/Xr15k5cyY///wzUVFR2Gw2EhMTiYyM/Nfn+eNmHF5eXnh7e3P58uV/vN7T0zOtaAEULVqUmJgYwDjsOjo6mpo1a6Z93Gq1Ur169dveev/YsWO4urpSq1attMcKFSpEhQoVOHbsGADdu3dn7NixbNy4kUaNGtGiRYu04tmlSxcGDRrE/v37ufvuu2nWrFlaaRMRERGRvEtlKwu4uLjwWb+GWbuMMOEGXl6eWbqM0NPTM934tddeY9OmTQwbNoyyZcvi4eHBoEGDSElJ+dfncXNzSzd2cXH512L05404svPA6H/y2GOPERYWxs8//8yvv/7Khx9+yLBhw+jWrRtNmjThp59+Yv369fz666/07NmTrl27MmzYMFMzi4iIiIi5TNsgI7dzcXHBy901C39Y//JYVm89v2vXLtq1a0fz5s0JDAzEz8+Pc+fOZenX/DMfHx/8/PzSnW1gs9nYv3//bT9nQEAAqamp7N69O+2xK1eucOLEibRdLQFKlChBly5dmDlzJr169eLTTz9N+1jhwoVp164d06ZNY8SIEXzyySe3nUdEREREcgfd2ZJbVq5cOdauXUvTpk1xcXFhxowZt71070488cQTfPDBB5QtW5aKFSuyaNEirl27dktl8/Dhw+TPnz9t7OLiQlBQEPfffz+jRo1i3LhxeHt7M23aNIoVK8b9998PwKuvvso999xD+fLliY2NZevWrQQEBADGBhnVq1encuXKJCcn8/PPP6d9TERERETyLpUtuWXDhw9nxIgRdO7cmUKFCtGnTx+uX7+e7Tn69OlDdHQ0w4YNw2q10rFjR8LCwm7pgLuuXbumG1utVvbv38/kyZN59dVX6devHykpKdStW5cPP/wwbQmk3W5n/PjxXLhwAW9vbxo3bszLL78MGMsk33jjDc6dO4eHhwd16tThjTfeyPwXLiIiIiI5iovD7DfD5AA2m43w8HBCQkL+8g/6xMRETpw4QYUKFfDw8MiWPMZ7thLw8vLK8qWDOYHdbqdly5a0bNmS5557zuw4WeJO59m/zWGRP9N8kYzSnJGM0pyRDLHbsa9/jWuHN+Hb6zOs7tnzb+5/kpH5qztbkuOcO3eOX3/9lXr16pGcnMzixYs5d+4cbdq0MTuaiIiIiGQmuw1WD8ayayEFsWBPiAH3UmanumUqW5LjWCwWVqxYwWuvvYbD4aBKlSrMnTtX75MSERERyU1sqbCyH0R8hsPFwslaQynrU9zsVBmisiU5TokSJVi2bJnZMUREREQkq6Qmw+dPwoHVYHHF3u5DLieXp+x/f6ZT0dbvIiIiIiLiPFJuwCddjaJldYdOi6DaI2anui26syUiIiIiIs4h+Tos7QwnfgFXT+iyBAKags1mdrLborIlIiIiIiLmS4yFxY/BmS3g7g2Pfwrl7zY71R1R2RIREREREXMlXIZF7SFyF3gUgCdWQOm6Zqe6YypbIiIiIiJinvgoWPgIXNwLXkWg2xdQopbZqTKFypaIiIiIiJgj9jwseBiiD4N3Mei+CopWNTtVptFuhHJHunXrxquvvpo2btq0KfPmzfvXzwkMDGTdunV3/LUz63lERERExARXT8PclkbR8i0Nvb7NVUULVLbyrH79+tG7d++//dhvv/1GYGAgBw8ezPDzLl++nE6dOt1pvHTeeecd2rZt+5fHN27cyD333JOpX+vPVqxYQd26OX+9sIiIiIhTiTkGH7eEKyegYDno9Q0UCTA7VaZT2cqjHn30UTZt2sSFCxf+8rHPP/+c4OBggoKCMvy8hQsXxtPTMzMi/id/f3/c3d2z5WuJiIiISCa5dBDmtoLYs1CkMjy5BgqVMztVllDZyqPuvfdeChcuzIoVK9I9fv36ddasWcOjjz7KlStXeOGFF2jcuDG1atWiTZs2fPXVV//6vH9eRnjy5Em6du1KjRo1aNWqFb/++utfPuf111/ngQceoFatWtx///3MmDGDlJQUwLizNHPmTA4ePEhgYCCBgYFpmf+8jPDQoUN0796dmjVr0qBBA0aNGsX169fTPj58+HD69+/PnDlzCAsLo0GDBowbNy7ta92OyMhInnnmGUJDQ6lduzaDBw8mOjo67eMHDx6kW7duaR9v3749ERERAJw7d45+/fpRr149QkJCaN26NevXr7/tLCIiIiJO7/wemNcK4i9A0erGHS3fkmanyjLaICOrOByQkpB1z518A1wd4OLy/8fdvNKP/4Wrqytt27bliy++4JlnnsHl989bs2YNdrudhx56iISEBKpXr06fPn3w9vbm559/ZujQoZQtW5aaNWv+59ew2+0MHDiQIkWK8NlnnxEXF8ekSZP+cl3+/PmZPHkyRYsW5fDhw4waNYr8+fPTp08fWrVqxZEjR9iwYQNz584FwMfH5y/PkZCQQO/evQkNDWX58uXExMQwcuRIJkyYwJQpU9Ku27p1K/7+/syfP5/Tp0/z/PPPU7VqVTp27HhLv25/fn39+/fHy8uLhQsXYrPZGDduHM8//zwLFy4EYMiQIVStWpWxY8di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\n"
          },
          "metadata": {}
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "- The training Loss decreases sharply at the beginning, which indicates that the model is quickly learning from the training data. As epochs increase, the rate of decrease slows down, suggesting that the model is starting to converge and is learning less from the training data with each epoch.\n",
        "- The validation Loss decreases along with the training loss, but it starts to plateau toward the end. The fact that the validation loss levels off but does not increase indicates that the model is not overfitting.\n",
        "- The early stopping after 30 epochs suggests that the model reached an optimal state in terms of generalization before performance on the validation set could deteriorate."
      ],
      "metadata": {
        "id": "YMMJoDL2huI4"
      }
    }
  ]
}
