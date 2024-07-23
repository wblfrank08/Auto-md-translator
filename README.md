# Auto-md-translator: An Automatic Multilingual Translation Tool Using ChatGPT

Auto-md-translator is a tool that automatically translates Markdown files into multiple languages using ChatGPT. You can achieve automatic translation locally using the `auto-translator.py` script, or utilize GitHub Actions to automatically translate into multiple languages (simply push the md files to the GitHub repository). (Modified from the original project: Auto-i18n) Currently supports translations from English to Chinese and Japanese.

Main features of Auto-md-translator:

- **Batch Multilingual Translation**: Auto-md-translator provides batch translation functionality, allowing you to translate all Markdown documents in an entire path into multiple languages (Chinese and Japanese).
- **Compatible with Front Matter**: Auto-md-translator is compatible with Markdown Front Matter syntax. You can customize translation or replacement rules for different fields.[NOT SURE WHAT THIS MEANS - Frank]
- **Fixed Content Replacement**: Auto-md-translator supports fixed content replacement. If you want certain recurring fields in the document to remain unchanged, this feature can help maintain consistency in your documents. [Haven't tried - Frank]
- **Automated Workflow**: You can use GitHub Actions to achieve an automated translation process. Translation work will be automatically performed and updated in the documents without manual intervention, allowing you to focus more on the content.

## Quick Start

1. Clone the repository to your local machine, rename `env_template.py` to `env.py`, and provide your ChatGPT API. If you don't have your own API, you can apply for a free one at [GPT_API_free](https://github.com/chatanywhere/GPT_API_free); alternatively, you can use the [go-chatgpt-api](https://github.com/linweiyuan/go-chatgpt-api) to convert the web version of ChatGPT to an API. Note that free APIs have a daily usage limit.
2. Install the necessary modules: `pip install -r requirements.txt`.
3. Run the program with the command `python auto-translater.py`. It will automatically process all Markdown files in the test directory `testdir/en-to-translate`, translating them into English, Spanish, and Arabic in batch. (More language support will be provided later)

## Detailed Description

The running logic of the `auto-translater.py` program is as follows:

1. The program will automatically process all Markdown files in the test directory `testdir/en-to-translate`. You can exclude files that don't need translation using the `exclude_list` variable.
2. Processed file names will be recorded in the automatically generated `processed_list.txt`. In the next run, already processed files will not be translated again.
3. If you need to retranslate specific articles (e.g., due to inaccurate translation or content changes), you can add the field `[translate]` to the article (leaving an empty line before and after). This will override the rules of `exclude_list` and `processed_list`, forcing translation processing.
4. If the Markdown file contains Front Matter, it will be processed according to the rules specified in `front_matter_translation_rules`:
   1. Automatic Translation: Translated by ChatGPT, suitable for fields like article titles or descriptions.
   2. Fixed Field Replacement: Suitable for fields like categories or tags to prevent inconsistent translations.
   3. No Processing: If the field is not listed in the above rules, it will remain unchanged, suitable for dates, URLs, etc.

## GitHub Actions Automation Guide

You can create a `.github/workflows/ci.yml` in your project repository. When updates are detected in the GitHub repository, GitHub Actions can automatically handle the translation and commit it back to the original repository.

The content of `ci.yml` can refer to the template: [ci_template.yml](https://github.com/linyuxuanlin/Auto-md-translator/blob/main/ci_template.yml).

You need to add two secrets in the repository's `Settings` - `Secrets and variables` - `Repository secrets`: `CHATGPT_API_BASE` and `CHATGPT_API_KEY`, and comment out the `import env` statement in the `auto-translater.py` program (or specify `auto-translater-github.py` in ci.yml).

## Troubleshooting

1. To verify the availability of the ChatGPT API key, you can use the [verify-api-key.py](https://github.com/linyuxuanlin/Auto-md-translator/blob/main/Archive/verify-api-key.py) program for testing. If using the official API in China, a local proxy is required.
2. If the Front Matter in the Markdown file cannot be recognized correctly, you can use the [detect_front_matter.py](https://github.com/linyuxuanlin/Auto-md-translator/blob/main/Archive/detect_front_matter.py) program for testing.
3. When encountering issues with GitHub Actions, first check if the path references are correct (e.g., `dir_to_translate`, `dir_translated_en`, `dir_translated_es`, `dir_translated_ar`, `processed_list`).

## Known Issues

1. In some special cases, there might be inaccurate translations or untranslated fields. It is recommended to manually review the translation before publishing the articles.

## Contribution

You are welcome to contribute to the improvement of this project! If you want to contribute code, report issues, or provide suggestions, please check out the [Contribution Guide](https://github.com/linyuxuanlin/Auto-md-translator/blob/main/CONTRIBUTING.md).

## License

This project is licensed under the [MIT License](https://github.com/linyuxuanlin/Auto-md-translator/blob/main/LICENSE).

## Issues and Support

If you encounter any issues while using Auto-md-translator or need technical support, feel free to [submit an issue](https://github.com/linyuxuanlin/Auto-md-translator/issues).

My blog uses Auto-md-translator to achieve multilingual support. You can check out the demo at [Power's Wiki](https://wiki-power.com).

[![](https://wiki-media-1253965369.cos.ap-guangzhou.myqcloud.com/img/202310222223670.png)](https://wiki-power.com)

## Acknowledgments

- Thanks to [chatanywhere/GPT_API_free](https://github.com/chatanywhere/GPT_API_free) for providing the free ChatGPT API key.
- Thanks to [linweiyuan/go-chatgpt-api](https://github.com/linweiyuan/go-chatgpt-api) for providing the method to convert the web version of ChatGPT to an API.

[![Star History Chart](https://api.star-history.com/svg?repos=linyuxuanlin/Auto-md-translator&type=Date)](https://star-history.com/#linyuxuanlin/Auto-md-translator&Date)
