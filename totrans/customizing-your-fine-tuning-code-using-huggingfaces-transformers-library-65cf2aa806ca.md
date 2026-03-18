# 使用 HuggingFace 的 Transformers 库自定义微调代码

> 原文：[`towardsdatascience.com/customizing-your-fine-tuning-code-using-huggingfaces-transformers-library-65cf2aa806ca/`](https://towardsdatascience.com/customizing-your-fine-tuning-code-using-huggingfaces-transformers-library-65cf2aa806ca/)

![由 Gemini 生成的图像](img/e9fec722de6b88b952d29ec080a579ba.png)

由 Gemini 生成的图像

HuggingFace 的 transformer 库提供了许多基本构建块和多种功能，以启动你的 AI 代码。许多产品和库都是基于它构建的，在这篇简短的博客中，我将讨论人们扩展它以在 HuggingFace transformer 库之上添加自定义训练代码的一些方法：

1.  **通过迭代训练数据来重新实现训练代码** 以重建微调循环，然后添加自定义代码，

1.  **创建附加到 _[训练器](https://huggingface.co/docs/transformers/en/main_classes/trainer)_ 类的自定义回调**，以便将自定义代码添加到回调中。

显然，可能还有其他自定义微调循环的方法，但本博客旨在关注这两种方法。

## 重新实现用于自定义微调循环的训练代码

通常在训练模型时，会创建一个 _[训练器](https://huggingface.co/docs/transformers/en/main_classes/trainer)_ 对象，该对象允许你指定训练模型的参数。训练器对象提供了一个 *train()* 方法，你可以调用它来启动训练循环：

```py
from transformers import AutoModelForCausalLM
from datasets import load_dataset
from trl import SFTConfig, SFTTrainer

dataset = load_dataset("stanfordnlp/imdb", split="train")
model = AutoModelForCausalLM.from_pretrained("facebook/opt-350m")

# Example Trainer object. SFTTrainer is types of trainer for supervised fine-tuning
trainer = SFTTrainer(
    model,
    train_dataset=dataset,
    args=SFTConfig(output_dir="/tmp"),
)
# Initiate training
trainer.train()
```

不同于调用训练器对象，一些库通过以下方式添加自定义代码：1) 在 *train()* 函数中重新实现传递数据以微调模型的代码，然后 2) 在重新实现的不同点添加自定义代码。一个很好的例子是 AllenAI 的开源库 [数据制图](https://github.com/allenai/cartography/tree/main)。

在库中，*训练动态* – 模型和训练数据点的特征 – 在微调循环的每个训练周期后都会被捕获。捕获训练动态需要在微调循环中编写自定义 [代码](https://github.com/allenai/cartography/blob/3df3438fcc7324e706dee2e787426389bbd8fb1a/cartography/classification/run_glue.py#L204-L264)。在代码中，创建了一个迭代器，它会遍历每个训练周期，并且对于每个训练周期，将训练数据的批次传递给模型进行训练（以下是对 [实现](https://github.com/allenai/cartography/blob/3df3438fcc7324e706dee2e787426389bbd8fb1a/cartography/classification/run_glue.py#L204-L264) 的简化、注释版本）：

```py
model.zero_grad()

# ------------------------------------------------------------------------------
# 1\. Creating an progress bar with trange (based on the tqdm library) 
#    to iterate through the specified epoch range
train_iterator = trange(epochs_trained, int(args.num_train_epochs), ...)
# ------------------------------------------------------------------------------

# --------------------- Add custom code here ------------------------------------
# ...
# ------------------------------------------------------------------------------

# Iterate through the epochs
for epoch, _ in enumerate(train_iterator):

  # Creating another iterator that goes through each of the batches of training data 
  # defined by the train_dataloader of type DataLoader object
  epoch_iterator = tqdm(train_dataloader, desc="Iteration", ...)

  # ------------------------------------------------------------------------------
  # 2\. Iterate through the batches of training data
  for step, batch in enumerate(epoch_iterator):
    # ------------------------------------------------------------------------------

    # --------------------- Add custom code here ------------------------------------
    # Such as checking if it is resuming a training loop and if so skipping past steps that have already been trained on
    # ------------------------------------------------------------------------------

    # Train the model
    model.train()
    # Prep the data according to the format expected by the model (this is a BERT model) 
    batch = tuple(t.to(args.device) for t in batch)
    inputs = {"input_ids": batch[0], "attention_mask": batch[1], "labels": batch[3]}
    outputs = model(**inputs)
    loss = outputs[0]

    # --------------------- Add custom code here ------------------------------------
    # Such as capturing the training dynamics, 
    # i.e. model and training data properties at that specified epoch
    if train_logits is None:  # Keep track of training dynamics.
        train_ids = batch[4].detach().cpu().numpy()
        train_logits = outputs[1].detach().cpu().numpy()
        train_golds = inputs["labels"].detach().cpu().numpy()
        train_losses = loss.detach().cpu().numpy()
    else:
        train_ids = np.append(train_ids, batch[4].detach().cpu().numpy())
        train_logits = np.append(train_logits, outputs[1].detach().cpu().numpy(), axis=0)
        train_golds = np.append(train_golds, inputs["labels"].detach().cpu().numpy())
        train_losses = np.append(train_losses, loss.detach().cpu().numpy())
    # ------------------------------------------------------------------------------ 
```

通过重新实现微调循环中执行的操作，**Trainer 对象的基本部分**（[`github.com/huggingface/trl/blob/v0.13.0/trl/trainer/base.py`](https://github.com/huggingface/trl/blob/v0.13.0/trl/trainer/base.py)）也被重新实现了，包括在训练数据的批次上执行训练步骤和计算数据批次上的模型损失。虽然这种自定义微调循环的方法让开发者对实现有更精细的控制，但这种方法也需要大量工作来确保代码能够正常工作。添加自定义代码的第二种方法不需要重新实现 Trainer 对象的部分，因为它使用*自定义回调*。

## **创建自定义回调**以自定义 Trainer 类

回调是一个作为参数传递给另一个函数的函数。第二个函数可以在稍后调用传递的函数。回调允许您在传递给第二个函数的函数中添加自定义代码。有一个[TrainerCallback](https://huggingface.co/docs/transformers/v4.47.1/en/main_classes/callback#transformers.TrainerCallback)类，它包含[空回调函数](https://github.com/huggingface/transformers/blob/241c04d36867259cdf11dbb4e9d9a60f9cb65ebc/src/transformers/trainer_callback.py#L260)，您可以用自定义代码覆盖它们。这些回调函数本质上是在 Trainer 类（或其继承版本，如 SFTTrainer，如果您在使用的话）的训练循环的不同部分调用的。

```py
# Taken straight from the TrainerCallback source code:
# https://github.com/huggingface/transformers/blob/v4.47.1/src/transformers/trainer_callback.py#L260
class TrainerCallback:
    # A bunch of empty functions you can override
    def on_train_begin(self, args: TrainingArguments, state: TrainerState, control: TrainerControl, **kwargs):
        """
        Event called at the beginning of training.
        """
        pass

    def on_train_end(self, args: TrainingArguments, state: TrainerState, control: TrainerControl, **kwargs):
        """
        Event called at the end of training.
        """
        pass

    def on_epoch_begin(self, args: TrainingArguments, state: TrainerState, control: TrainerControl, **kwargs):
        """
        Event called at the beginning of an epoch.
        """
        pass

    def on_epoch_end(self, args: TrainingArguments, state: TrainerState, control: TrainerControl, **kwargs):
        """
        Event called at the end of an epoch.
        """
        pass
    # More empty functions not included here...
```

每个空函数都传递了几个参数（1）*args*，类型为 TrainingArguments，（2）*state*，类型为 TrainerState，（3）*control*，类型为 TrainerControl，以及（4）额外的任意参数，这些参数组合成*kwargs*参数。这些参数包含当前对 Trainer 类有效的对象，您可以访问并添加自定义代码。关于这些参数的更多详细信息可以在[这里](https://huggingface.co/docs/transformers/v4.47.0/en/main_classes/callback#transformers.TrainerCallback)找到。

例如，如果您想在每个 epoch 结束后访问模型的当前状态，您会

1.  覆盖 _on_epoch_end_ 函数，并

1.  在 _on_epoch_end_ 函数中访问模型

```py
from transformers import TrainerCallback, TrainerState, TrainerControl

class ExampleTrainerCallback(TrainerCallback):
    """Custom ExampleTrainerCallback that accesses the model after each epoch
    """
    def __init__(self, some_tokenized_dataset):
        """Initializes the ExampleTrainerCallback instance."""
        super().__init__()
        # --------------------- Add custom code here ------------------------------------
        self.some_tokenized_dataset = some_tokenized_dataset
        # ------------------------------------------------------------------------------

    # Overriding the on_epoch_end() function
    def on_epoch_end(self, args: TrainingArguments, state: TrainerState, control: TrainerControl, **kwargs):
        """
        Event called at the end of an epoch.
        """
        # --------------------- Add custom code here ------------------------------------
        print('Hello an epoch has ended!')

        # Access the current state of the model after the epoch ends: 
        model = kwargs["model"]

        # Add some custom code here...
        model.eval()

        # Perform inference on some dataset
        with torch.no_grad():
            for item in self.some_tokenized_dataset: 
                input_ids = item["input_ids"].unsqueeze(0)  # Add batch dimension
                attention_mask = item["attention_mask"].unsqueeze(0)  # Add batch dimension

                # Forward pass, assuming model is a BertForSequenceClassification type
                # i.e. model = BertForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)  
                outputs = model(input_ids=input_ids, attention_mask=attention_mask)
                logits = outputs.logits
                probabilities = torch.nn.functional.softmax(logits, dim=-1)
                prediction = torch.argmax(probabilities, dim=-1).item()
                # Do something with prediction
        # ------------------------------------------------------------------------------
```

在上面的代码中，我们使用*kwargs["model"]*行在每个 epoch 结束后访问模型的当前状态，然后我们添加一些自定义代码（在这种情况下，我们在 ExampleTrainerCallback 类的 init 函数中对一些数据集进行 tokenization 后进行推理）。使用 TrainerCallback 的好处是，我们不需要深入实现 Trainer 类中的一些核心操作，包括计算损失和将训练数据的批次传递到模型中进行学习。我们通过使用现有的代码（其他人已经测试过的代码）来保留所有这些操作，并在现有代码的基础上添加我们的自定义代码。

自定义回调的一个主要问题是确保您也将它传递到您将要使用的 Trainer 对象中：

```py
from transformers import BertTokenizer, BertForSequenceClassification, Trainer, TrainingArguments
from datasets import load_dataset
from trl import SFTTrainer

# Create a Trainer Object
trainer = SFTTrainer(
    model=model,
    train_dataset=train_data,
    args=TrainingArguments(num_train_epochs=5, evaluation_strategy='epoch', ...),
    # Additional arguments here...
)

tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
# Prep some data, where some_tokenized_dataset is of type DatasetDict
def tokenize_function(example):
    return tokenizer(example['text'], padding="max_length", truncation=True)
some_tokenized_dataset = load_dataset('json', data_files='path_to_your_data', split='test')
some_tokenized_dataset = some_tokenized_dataset.map(tokenize_function, batched=True)

# --------------- DO NOT FORGET TO ADD YOUR CALLBACKS TO YOUR TRAINER! -----------------------------------------------
# Create the callback with you custom code
example_callback = ExampleTrainerCallback(
  some_tokenized_dataset = some_tokenized_dataset
)

# Add the callback to the Trainer
trainer.add_callback(example_callback)

# ------------------------------------------------------------------------------

# Train the model
trainer.train()
```

为了说明一个具体的例子，[Weights and Biases](https://wandb.ai/site/) 库有一个名为 [WandbCallback](https://github.com/huggingface/transformers/blob/v4.47.1/src/transformers/integrations/integration_utils.py#L759) 的示例 TrainerCallback，它在微调循环期间添加自定义代码。WandbCallback 在训练循环中记录了诸如指标和模型检查点等信息，然后会将这些信息发送到 Weights and Biases，以便您稍后可以使用他们的工具（以下是对 [自定义回调](https://github.com/huggingface/transformers/blob/v4.47.1/src/transformers/integrations/integration_utils.py#L759) 的简化、注释版本）：

```py
from transformers import TrainerCallback

class WandbCallback(TrainerCallback):
    """
    A [`TrainerCallback`] that logs metrics, media, model checkpoints to [Weight and Biases](https://www.wandb.com/).
    """

    def __init__(self):
        # ---------------------------------------------------------------------------
        # Custom code that runs when the WandbCallback class is initialized
        # ---------------------------------------------------------------------------

    def on_train_end(self, args, state, control, model=None, tokenizer=None, **kwargs):
        # ---------------------------------------------------------------------------
        # Custom code 
        # add the model architecture to a separate text file
        save_model_architecture_to_file(model, temp_dir)
        # ---------------------------------------------------------------------------

    def on_log(self, args, state, control, model=None, logs=None, **kwargs):
        # ---------------------------------------------------------------------------
        # Custom code that logs the items listed in single_value_scalars
        single_value_scalars = [
            "train_runtime",
            "train_samples_per_second",
            "train_steps_per_second",
            "train_loss",
            "total_flos",
        ]

        # More code here that accesses these values in the logs argument that then gets saved to wandb
        if state.is_world_process_zero:
            for k, v in logs.items():
                if k in single_value_scalars:
                    self._wandb.run.summary[k] = v
            non_scalar_logs = {k: v for k, v in logs.items() if k not in single_value_scalars}
            non_scalar_logs = rewrite_logs(non_scalar_logs)
            self._wandb.log({**non_scalar_logs, "train/global_step": state.global_step})
        # --------------------------------------------------------------------------- 
```

**旁注：** 使用 TrainingArgument 类进行微调时（该类随后传递给 Trainer 类），Weights and Biases 有时默认启用。如果您使用专有数据（也取决于您所在组织的政策），则应将其禁用：

```py
from transformers.training_args import TrainingArguments
from trl import SFTTrainer

# Disable any reporting of internal, proprietary data when using HuggingFace classes:
args = TrainingArguments(report_to=None, ...)

trainer = SFTTrainer(
    model=model,
    train_dataset=train_data,
    args=args
)
```

## 结论

这篇简短的博客介绍了在 HuggingFace Transformers 库中自定义微调循环的一些方法。我借鉴了现有库和代码中的两个不同示例，并讨论了每种方法的优缺点。应该还存在许多其他创造性的方法，但它们超出了这篇迷你博客的范围。
