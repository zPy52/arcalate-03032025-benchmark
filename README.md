# Human evaluation benchmarks

Arcalate shines in nuanced and heavily context-based tasks, when the process of finding the right translation becomes difficult even for a professional human being. 
This is precisely the biggest obstacle for AI-based translators because they cannot think their way through the many possible valid translations that one can come up with for a given sentence.
For example, in Spanish `¿Qué pasa primo?` is an idiomatic expression that means something like `What's up, bro?`. But this very sentence has many synonyms:

- *What's up, man?*
- *What's up, dude?*
- *What's up, mate?*
- *What's up, buddy?*
- ...

Everyone knows that current tools like Google Translate and DeepL have a very hard time trying to get these kind of translations right. But the problem we face here is to quantify how hard it is for them.
How many times does Google Translate fail? And DeepL? How can we assess their outputs at scale in a rigorous manner?

Current metrics are not sufficient to really assess the effectiveness of these models. BLEU (Bilingual Evaluation Understudy) is the standard to compare performance between translations and it is very convinient to implement in code... but it falls short when it comes to comparing real-life use cases quality.
BLEU calculates the n-gram precision, which measures the overlap of n-word sequences between the machine-generated translation and the reference translations. It also incorporates a brevity penalty to discourage overly short translations.
Its most blatant caveat is its rigidity: only exact n-gram matches are valid and BLEU does not account for the meaning behind the words. Consequently, a translation might receive a low BLEU score even if it accurately conveys the intended meaning using different wording. [Spot Intelligence](https://spotintelligence.com/2024/08/13/bleu-score-in-nlp/)

Thus, since we really only care about the model's prowess to accurately adapt to the context and make sensitive, natural, human-like translations, we are in the need for a non-programmatic benchmark that compares their effectiveness.

With this in mind, we have concocted a dataset of 236 Spanish to English non-literal translations. These are some examples:

```json
[
  {"es": "¿Qué pasa primo?", "en": "What's up, bro?"},
  {"es": "No ver la hora", "en": "To be in a rush."},
  {"es": "A otro perro con ese hueso", "en": "I don't buy that nonsense."},
  {"es": "Salir airoso", "en": "To come out unscathed."},
  {"es": "Estar que trina", "en": "To be buzzing."},
  ...
]
```

These were recollected from looking on the Internet, as they are relatively easy to find. From these, we have provided the Spanish source (every `spa_Latn` instance) to Google Translate, DeepL and both Arcalate and Arcalate with Deep Translate active.
The results were then given one of these three scores:

- 0: if the translation was wrong,
- 0.5: if it was mostly right,
- 1: if it correctly grasped the meaning and context, resulting in a valid non-literal translation.

These were assigned by human evaluators internal to the project. Here are some examples of both the translations and their evaluations:

```json
[
  {
    "source": "Poner toda la carne en el asador",
    "reference": "To go all out.",
    "translations": {
      "DeepL": "Putting all the meat on the grill",
      "Google Translate": "Put all the meat on the grill",
      "Arcalate": "Put all your eggs in one basket.",
      "Arcalate (Deep Translate)": "Go all out"
    },
    "assessment": {
      "DeepL": 0,
      "Google Translate": 0,
      "Arcalate": 0,
      "Arcalate (Deep Translate)": 1
    }
  },
  {
    "source": "Estar en la luna",
    "reference": "To have one's head in the clouds.",
    "translations": {
      "DeepL": "Being on the moon",
      "Google Translate": "Be on the moon",
      "Arcalate": "To be daydreaming",
      "Arcalate (Deep Translate)": "to have one's head in the clouds"
    },
    "assessment": {
      "DeepL": 0,
      "Google Translate": 0,
      "Arcalate": 0.5,
      "Arcalate (Deep Translate)": 1
    }
  },
  {
    "source": "Ser un manitas",
    "reference": "To be a real handyman.",
    "translations": {
      "DeepL": "Being a handyman",
      "Google Translate": "Be a handyman",
      "Arcalate": "to be handy",
      "Arcalate (Deep Translate)": "To be handy"
    },
    "assessment": {
      "DeepL": 1.0,
      "Google Translate": 1.0,
      "Arcalate": 0.5,
      "Arcalate (Deep Translate)": 0.5
    }
  },
  {
    "source": "Echar agua al mar",
    "reference": "To do something pointless.",
    "translations": {
      "DeepL": "Pouring water into the sea",
      "Google Translate": "Throw water",
      "Arcalate": "To add water to the ocean",
      "Arcalate (Deep Translate)": "to add a drop to the ocean"
    },
    "assessment": {
      "DeepL": 0,
      "Google Translate": 0,
      "Arcalate": 0,
      "Arcalate (Deep Translate)": 0
    }
  },
  {
    "source": "Ver las estrellas",
    "reference": "To be knocked senseless.",
    "translations": {
      "DeepL": "See the stars",
      "Google Translate": "See stars",
      "Arcalate": "To see the stars",
      "Arcalate (Deep Translate)": "to dream big"
    },
    "assessment": {
      "DeepL": 0,
      "Google Translate": 0,
      "Arcalate": 0,
      "Arcalate (Deep Translate)": 0
    }
  },
  ...
]
```

After careful and rigorous evaluation of every instance, we have summed up the total scores and divided by $n = 236$. This results in:

1. Arcalate (Deep Translate): $95.13$% accuracy.
2. Arcalate: $77.54$% accuracy.
3. DeepL: $50.64$% accuracy.
4. Google Translate: $17.16$% accuracy.

It shall be noted that the dataset contains non-literal expressions **only**, which ignores the capabilities of the four models in easier, more literal translation setups. We have purposefully done this because in general these models do well on tasks of little creative, cultural, and lexical complexity. We presume that smart models capable of adapting to context and taking into account cultural nuances shall in general perform well in simpler projects due to its very intelligence.
