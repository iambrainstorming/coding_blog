# Open TTS

```bash
docker run -it -p 5500:5500 synesthesiam/opentts:en
```

<https://github.com/synesthesiam/opentts>


Visit <http://localhost:5500>

For HTTP API test page, visit <http://localhost:5500/openapi/>

```xml
<speak>
  Sentences outside any tags are spoken by the selected voice.
  <s>
    You can manually separate sentences <w>and</w><w>words</w>.
  </s>

  <voice name="coqui-tts:en_vctk#p228">
    <s>
      The current voice can be changed, even to a different text to speech system!
    </s>
  </voice>

  <voice name="coqui-tts:en_ljspeech">
    <s>
      Default voices exist for all of the supported languages.
    </s>
  </voice>

  <voice name="coqui-tts:en_vctk#p228">
    <s>Breaks are possible</s>
    <break time="0.5s" />
    <s>between sentences.</s>
  </voice>
</speak>
```
