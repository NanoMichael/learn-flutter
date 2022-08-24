## Dart 入口：

- file: lib/ui/text/paragraph_builder.h

```c++
namespace flutter {
class ParagraphBuilder : public RefCountedDartWrappable<ParagraphBuilder> {
  DEFINE_WRAPPERTYPEINFO();
  FML_FRIEND_MAKE_REF_COUNTED(ParagraphBuilder);

 public:
  static void Create(Dart_Handle wrapper,
                     Dart_Handle encoded_handle,
                     Dart_Handle strutData,
                     const std::string& fontFamily,
                     const std::vector<std::string>& strutFontFamilies,
                     double fontSize,
                     double height,
                     const std::u16string& ellipsis,
                     const std::string& locale);
 private:
  explicit ParagraphBuilder(Dart_Handle encoded,
                            Dart_Handle strutData,
                            const std::string& fontFamily,
                            const std::vector<std::string>& strutFontFamilies,
                            double fontSize,
                            double height,
                            const std::u16string& ellipsis,
                            const std::string& locale);

  std::unique_ptr<txt::ParagraphBuilder> m_paragraphBuilder;
}
```

通过将 Dart 侧配置转换为 C++ style：

- file: lib/ui/text/paragraph_builder.cc

```c++
ParagraphBuilder::ParagraphBuilder(
    Dart_Handle encoded_data,
    Dart_Handle strutData,
    const std::string& fontFamily,
    const std::vector<std::string>& strutFontFamilies,
    double fontSize,
    double height,
    const std::u16string& ellipsis,
    const std::string& locale) {
// ...

#if FLUTTER_ENABLE_SKSHAPER
#if FLUTTER_ALWAYS_USE_SKSHAPER
  bool enable_skparagraph = true;
#else
  bool enable_skparagraph = UIDartState::Current()->enable_skparagraph();
#endif
  if (enable_skparagraph) {
    factory = txt::ParagraphBuilder::CreateSkiaBuilder;
  }
#endif  // FLUTTER_ENABLE_SKSHAPER

  m_paragraphBuilder = factory(style, font_collection.GetFontCollection());
}
```

`FLUTTER_ENABLE_SKSHAPER` 是否使用 skia 的 text layout，3.1.0 开始默认使用 skia。

## skia

- file: third_party/txt/src/txt/paragraph_builder.cc

```c++
#if FLUTTER_ENABLE_SKSHAPER

std::unique_ptr<ParagraphBuilder> ParagraphBuilder::CreateSkiaBuilder(
    const ParagraphStyle& style,
    std::shared_ptr<FontCollection> font_collection) {
  return std::make_unique<ParagraphBuilderSkia>(style, font_collection);
}

#endif  // FLUTTER_ENABLE_SKSHAPER
```

- file: third_party/txt/src/skia/paragraph_builder_skia.h

```C++
class ParagraphBuilderSkia : public ParagraphBuilder {
 public:
  ParagraphBuilderSkia(const ParagraphStyle& style,
                       std::shared_ptr<FontCollection> font_collection);

  virtual ~ParagraphBuilderSkia();

  virtual void PushStyle(const TextStyle& style) override;
  virtual void Pop() override;
  virtual const TextStyle& PeekStyle() override;
  virtual void AddText(const std::u16string& text) override;
  virtual void AddPlaceholder(PlaceholderRun& span) override;
  virtual std::unique_ptr<Paragraph> Build() override;

 private:
  std::shared_ptr<skia::textlayout::ParagraphBuilder> builder_;
  TextStyle base_style_;
  std::stack<TextStyle> txt_style_stack_;
};
```

里边又套了一个 `builder_`，类型为 `skia::textlayout::ParagraphBuilder`，仅仅是做了一层代理，为了适配 `txt` 的 API。

- file: third_party/skia/modules/skparagraph/include/ParagraphBuilder.h

```c++
class ParagraphBuilder {
public:
    ParagraphBuilder(const ParagraphStyle&, sk_sp<FontCollection>) { }

    virtual ~ParagraphBuilder() = default;

    // Push a style to the stack. The corresponding text added with AddText will
    // use the top-most style.
    virtual void pushStyle(const TextStyle& style) = 0;
    
    // ...
    // Just until we fix all the google3 code
    static std::unique_ptr<ParagraphBuilder> make(const ParagraphStyle& style,
                                                  sk_sp<FontCollection> fontCollection);
}
```

就是重复了 `txt::ParagraphBuilder` 那一套接口。

- file: <ENGINE_ROOT>/engine/src/third_party/skia/modules/skparagraph/src/ParagraphBuilderImpl.cpp

```c++
namespace skia {
namespace textlayout {

std::unique_ptr<ParagraphBuilder> ParagraphBuilder::make(
        const ParagraphStyle& style, sk_sp<FontCollection> fontCollection) {
    return ParagraphBuilderImpl::make(style, fontCollection);
}

std::unique_ptr<ParagraphBuilder> ParagraphBuilderImpl::make(
        const ParagraphStyle& style, sk_sp<FontCollection> fontCollection) {
    auto unicode = SkUnicode::Make();
    if (nullptr == unicode) {
        return nullptr;
    }
    return std::make_unique<ParagraphBuilderImpl>(style, fontCollection);
}

std::unique_ptr<Paragraph> ParagraphBuilderImpl::Build() {
    if (!fUtf8.isEmpty()) {
        this->endRunIfNeeded();
    }

    // Add one fake placeholder with the rest of the text
    addPlaceholder(PlaceholderStyle(), true);
    return std::make_unique<ParagraphImpl>(
            fUtf8, fParagraphStyle, fStyledBlocks, fPlaceholders, fFontCollection, fUnicode);
}

}
```