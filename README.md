{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "name": "VoiceChangerDemo",
      "provenance": [],
      "authorship_tag": "ABX9TyOby01VR127ZMS2YWFP8/YE",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    },
    "accelerator": "GPU",
    "gpuClass": "standard"
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/github/w-okada/voice-changer/blob/dev/VoiceChangerDemo.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "MMVCプレイヤー（普通版）\n",
        "---\n",
        "\n",
        "このノートはColab上でMMVCのボイチェンを行うノートです。\n",
        "\n",
        "正式版はローカルPC上で動かすアプリケーションです。\n",
        "\n",
        "正式版は、多くの場合より少ないタイムラグで滑らかに音声を変換できます。\n",
        "\n",
        "詳細な使用方法はこちらの[リポジトリ](https://github.com/w-okada/voice-changer)からご確認ください。\n"
      ],
      "metadata": {
        "id": "Lbbmx_Vjl0zo"
      }
    },
    {
      "cell_type": "markdown",
      "source": [
        "# GPUを確認\n",
        "GPUを用いたほうが高速に処理が行えます。\n",
        "\n",
        "下記のコマンドでGPUが確認できない場合は、上のメニューから\n",
        "\n",
        "「ランタイム」→「ランタイムの変更」→「ハードウェア アクセラレータ」\n",
        "\n",
        "でGPUを選択してください。"
      ],
      "metadata": {
        "id": "oUKi1NYMmXrr"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (1) GPUの確認\n",
        "!nvidia-smi"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "vV1t7PBRm-o6",
        "outputId": "de727091-7c9f-4dbc-a0cc-5985409b289f"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Sun Jan 29 12:09:35 2023       \n",
            "+-----------------------------------------------------------------------------+\n",
            "| NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |\n",
            "|-------------------------------+----------------------+----------------------+\n",
            "| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |\n",
            "| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |\n",
            "|                               |                      |               MIG M. |\n",
            "|===============================+======================+======================|\n",
            "|   0  Tesla T4            Off  | 00000000:00:04.0 Off |                    0 |\n",
            "| N/A   47C    P0    26W /  70W |      0MiB / 15360MiB |      0%      Default |\n",
            "|                               |                      |                  N/A |\n",
            "+-------------------------------+----------------------+----------------------+\n",
            "                                                                               \n",
            "+-----------------------------------------------------------------------------+\n",
            "| Processes:                                                                  |\n",
            "|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |\n",
            "|        ID   ID                                                   Usage      |\n",
            "|=============================================================================|\n",
            "|  No running processes found                                                 |\n",
            "+-----------------------------------------------------------------------------+\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# 使用するモデルとコンフィグファイルの指定\n",
        "\n",
        "使用するトレーニング済みのモデルと、トレーニングで使用したコンフィグファイルのパスを指定してください。\n",
        "\n",
        "多くの場合はGoogle Driveに格納されているファイルを使用すると思います。その場合は、下の(2-2)のセルを実行してドライブをマウントしてください"
      ],
      "metadata": {
        "id": "mHvGrgaWnIPA"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (2-1) 使用するモデルとコンフィグファイルの指定\n",
        "if \"MODEL\" in locals():\n",
        "  del MODEL\n",
        "if \"ONNX\" in locals():\n",
        "  del ONNX\n",
        "\n",
        "CONFIG=\"/content/drive/MyDrive/VoiceChanger/config.json\"\n",
        "#MODEL=\"/content/drive/MyDrive/VoiceChanger/G_326000.pth\"\n",
        "ONNX=\"/content/drive/MyDrive/VoiceChanger/G_326000.onnx\""
      ],
      "metadata": {
        "id": "nSXATMWYb4Ik"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "2wxD-gRSMU5R",
        "outputId": "4c9da537-a5cb-4c5d-f999-2795b00d41a8"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Mounted at /content/drive\n"
          ]
        }
      ],
      "source": [
        "# (2-2) Google Driveのマウント\n",
        "from google.colab import drive\n",
        "drive.mount('/content/drive')"
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# リポジトリのクローン\n",
        "リポジトリをクローンします"
      ],
      "metadata": {
        "id": "sLBfykjBnjWc"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (3) リポジトリのクローン\n",
        "!git clone --depth 1 https://github.com/w-okada/voice-changer.git  -b v.1.3.7\n",
        "%cd voice-changer/server\n",
        "!git clone https://github.com/isletennos/MMVC_Client.git\n",
        "!cd MMVC_Client && git checkout 04f3fec4fd82dea6657026ec4e1cd80fb29a415c && cd -"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "86wTFmqsNMnD",
        "outputId": "2b329892-41b2-4560-e4f0-ab49b4ef83bc"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Cloning into 'voice-changer'...\n",
            "remote: Enumerating objects: 157, done.\u001b[K\n",
            "remote: Counting objects: 100% (157/157), done.\u001b[K\n",
            "remote: Compressing objects: 100% (141/141), done.\u001b[K\n",
            "remote: Total 157 (delta 21), reused 70 (delta 5), pack-reused 0\u001b[K\n",
            "Receiving objects: 100% (157/157), 1.61 MiB | 4.03 MiB/s, done.\n",
            "Resolving deltas: 100% (21/21), done.\n",
            "/content/voice-changer/server\n",
            "Cloning into 'MMVC_Client'...\n",
            "remote: Enumerating objects: 611, done.\u001b[K\n",
            "remote: Counting objects: 100% (337/337), done.\u001b[K\n",
            "remote: Compressing objects: 100% (120/120), done.\u001b[K\n",
            "remote: Total 611 (delta 238), reused 285 (delta 214), pack-reused 274\u001b[K\n",
            "Receiving objects: 100% (611/611), 752.38 KiB | 21.50 MiB/s, done.\n",
            "Resolving deltas: 100% (360/360), done.\n",
            "Note: switching to '04f3fec4fd82dea6657026ec4e1cd80fb29a415c'.\n",
            "\n",
            "You are in 'detached HEAD' state. You can look around, make experimental\n",
            "changes and commit them, and you can discard any commits you make in this\n",
            "state without impacting any branches by switching back to a branch.\n",
            "\n",
            "If you want to create a new branch to retain commits you create, you may\n",
            "do so (now or later) by using -c with the switch command. Example:\n",
            "\n",
            "  git switch -c <new-branch-name>\n",
            "\n",
            "Or undo this operation with:\n",
            "\n",
            "  git switch -\n",
            "\n",
            "Turn off this advice by setting config variable advice.detachedHead to false\n",
            "\n",
            "HEAD is now at 04f3fec Merge pull request #30 from Mokuichi147/setupcheck\n",
            "/content/voice-changer/server\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# モジュールのインストール\n",
        "\n",
        "必要なモジュールをインストールします。"
      ],
      "metadata": {
        "id": "8Na2PbLZSWgZ"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (5) 設定ファイルの確認\n",
        "!apt-get install -y libsndfile1-dev &> /dev/null\n",
        "!pip install fastapi &> /dev/null\n",
        "!pip install pyOpenSSL &> /dev/null\n",
        "!pip install python-multipart &> /dev/null\n",
        "!pip install python-socketio &> /dev/null\n",
        "!pip install uvicorn &> /dev/null\n",
        "!pip install websockets &> /dev/null\n",
        "!pip install onnxruntime-gpu &> /dev/null"
      ],
      "metadata": {
        "id": "LwZAAuqxX7yY"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "# サーバの起動\n",
        "\n",
        "サーバを起動します。(6-1)\n",
        "\n",
        "サーバの起動状況を確認します。(6-2) \n",
        "\n",
        "このセルは繰り返し実行することになるのでCtrl+Retでセルを実行してください。\n",
        "\n",
        "アクセスできるようになるまで、１～２分かかるようです。コーヒーでも飲みに行きましょう。\n",
        "\n",
        "下記のようなテキストが表示されたら起動完了です。\n",
        "(warningは無視して問題ありません。)\n",
        "```\n",
        "/usr/local/lib/python3.8/dist-packages/onnxruntime/capi/onnxruntime_inference_collection.py:54: UserWarning: Specified provider 'OpenVINOExecutionProvider' is not in available provider names.Available providers: 'TensorrtExecutionProvider, CUDAExecutionProvider, CPUExecutionProvider'\n",
        "  warnings.warn(\n",
        "/usr/local/lib/python3.8/dist-packages/onnxruntime/capi/onnxruntime_inference_collection.py:54: UserWarning: Specified provider 'DmlExecutionProvider' is not in available provider names.Available providers: 'TensorrtExecutionProvider, CUDAExecutionProvider, CPUExecutionProvider'\n",
        "  warnings.warn(\n",
        "VoiceChanger Initialized (GPU_NUM:1, mps_enabled:False)\n",
        "    Voice Changerを起動しています。\n",
        "    -- 設定 -- \n",
        "    CONFIG:/content/drive/MyDrive/VoiceChanger/config.json, MODEL:None ONNX_MODEL:/content/drive/MyDrive/VoiceChanger/G_326000.onnx\n",
        "```\n",
        "\n"
      ],
      "metadata": {
        "id": "-_2OcN9Borke"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (6-1) サーバの起動\n",
        "import random\n",
        "PORT = 10000 + random.randint(1, 9999)\n",
        "LOG_FILE = f\"LOG_FILE_{PORT}\"\n",
        "\n",
        "if \"MODEL\" in locals() and \"ONNX\" in locals():\n",
        "  model_param = f\" -m {MODEL} -o {ONNX}\"\n",
        "elif \"MODEL\" in locals():\n",
        "  model_param = f\" -m {MODEL}\"\n",
        "elif \"ONNX\" in locals():\n",
        "  model_param = f\" -o {ONNX}\"\n",
        "else:\n",
        "  model_param = f\"\"\n",
        "\n",
        "get_ipython().system_raw(f'python3 MMVCServerSIO.py -t MMVC -p {PORT} -c {CONFIG} {model_param}  --https False --colab True >{LOG_FILE} 2>&1 &')\n",
        "#print(f\"PORT:{PORT}, LOG_FILE:{LOG_FILE}\")"
      ],
      "metadata": {
        "id": "iNOAB7zISI6J"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# (6-2) サーバの起動確認 (Ctrl+Retで実行)\n",
        "!tail -20 {LOG_FILE}"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "chu06KpAjEK6",
        "outputId": "cf0d26f3-66a9-406e-e739-f588483d2851"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "/usr/local/lib/python3.8/dist-packages/onnxruntime/capi/onnxruntime_inference_collection.py:54: UserWarning: Specified provider 'OpenVINOExecutionProvider' is not in available provider names.Available providers: 'TensorrtExecutionProvider, CUDAExecutionProvider, CPUExecutionProvider'\n",
            "  warnings.warn(\n",
            "/usr/local/lib/python3.8/dist-packages/onnxruntime/capi/onnxruntime_inference_collection.py:54: UserWarning: Specified provider 'DmlExecutionProvider' is not in available provider names.Available providers: 'TensorrtExecutionProvider, CUDAExecutionProvider, CPUExecutionProvider'\n",
            "  warnings.warn(\n",
            "VoiceChanger Initialized (GPU_NUM:1, mps_enabled:False)\n",
            "\u001b[32m    Voice Changerを起動しています。\u001b[0m\n",
            "\u001b[34m    -- 設定 -- \u001b[0m\n",
            "\u001b[34m    CONFIG:/content/drive/MyDrive/VoiceChanger/config.json, MODEL:None ONNX_MODEL:/content/drive/MyDrive/VoiceChanger/G_326000.onnx\u001b[0m\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "# プロキシを起動\n",
        "ウェブサーバへのアクセスをするためのプロキシを起動します。\n",
        "\n",
        "表示されたURLをクリックして開くと別タブでアプリが開きます。\n",
        "\n",
        "Colabなので、ロードにある程度時間がかかります(30秒くらい)。"
      ],
      "metadata": {
        "id": "WhxcFLQEpctq"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# (7) プロキシを起動\n",
        "from google.colab.output import eval_js\n",
        "proxy = eval_js( \"google.colab.kernel.proxyPort(\" + str(PORT) + \")\" )\n",
        "print(f\"{proxy}front/?colab=true\")"
      ],
      "metadata": {
        "id": "nkRjZm95l87C",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 34
        },
        "outputId": "b6ef1db3-37b4-4025-fdbf-711e00646902"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "https://nafdn2dmorn-496ff2e9c6d22116-18341-colab.googleusercontent.com/front/?colab=true\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [],
      "metadata": {
        "id": "Jos5WZHGmz4s"
      },
      "execution_count": null,
      "outputs": []
    }
  ]
}
