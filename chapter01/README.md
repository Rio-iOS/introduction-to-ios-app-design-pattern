# 設計を知る

## 設計するということ

### アプリで可能なことの増加
- iOSアプリケーションでは、起動経路が多様化し、デバイスで取得できる情報の種類も増加
- 端末のレイアウトも多様化
    - (ex)
    - 320 × 480、ビクセルとポイントが1対1で紐づく時代は終了
    - iPhoneXの登場以降、Safe Areaの概念も登場
- **どのような機種でも破綻なくレイアウトを成立させる必要がある**
- **ユーザーインタラクションへの細かい反応も要求される**
    - (ex)
    - WWDC2018のセッションで、トランジションの途中であっても中断・復帰可能なインタラクションが推奨
- **アプリで起こりうる状態のパターンを洗い出し、網羅的に対処できるコードになっていなければ、想定外のバグを引き起こす**

### 継続的なリリース
- **近年のアプリは、頻繁かつ継続的なリリースを要求される**
- **リリースサイクルを短縮することにはさまざまなメリットがある**
    - (ex)
    - 競合アプリに先駆けて機能を投入
    - ユーザーの反応をもとにきめ細かやかなチューニング可能
- **小幅な改修をユーザーにさらすことは不具合のリスクを最小化する**
- **改修によってバグを作り込むことは避けなければいけない**
- リグレッションテスト：既存の挙動を保証すること
- **人の手によるテストは非効率であり、ミスを生みがち**
- **自動テストはどんなコードに対しても行えるものではない**
- **テストコードを前提としたコードを書かなければ、アプリの品質は保てず、継続的なリリースに師匠が生じる**

### アプリの大規模化
- ひとつのアプリで、画面数もユーザー数も増加
- アプリ内のデータやデザインの整合性を保つ労力は、その画面数に応じて増大
- **アプリをスケールさせるためには、それぞれの画面でコンポーネントやロジックを再利用できるようにしなければいけない**
- **プロジェクトのコードは似たようで微妙に違うコピペの山となり、管理不能になる**
- ユーザーが増えれば、バグのインパクトも大きくなる
    - Crashlyticsなどのクラッシュログ情報ツール上のクラッシュフリレートをどれだけ上げられるかでアプリの評価は大きく変わる
- **いかに迅速で不具合を見つけられるか、あるいは不具合が起こらないようなしくみを作るかが重要**

### プロジェクトの最適化
- **アプリを作ったなら、ユーザには長く使われたいと思う**
- **長い歴史の中で、開発コミュニティの環境も移り変わる**
- **利用するライブラリやフレームワークの改修への対応も起こる**
    - (ex)
    - 永続化フレームワークをCore DataからRealmに移行
    - 言語自体をObjective-CからSwiftに書き換え
- **特定のフレームワークを前提としたコードは移行が難しかったり、移行が致命的なバグを生んでしまったり、最悪の場合は移行不可能と判断される**
- **サポートが打ち切られセキュリティリスクを抱えるライブラリを不債として使い続けられるような事態は起こしたくない**
- **ライブラリはあくまでも開発を支援するためのもの**
- **特定のライブラリを使いたくて、アプリを開発しているわけではない**
- **環境が変わっても、その影響を最小限にとどめ、なるべく本来やりたい機能開発に集中できるような「変化を織り込んだ」構造を整備する必要がある**

### チーム開発での作業
- **ひとつのアプリを多人数のチームで開発する状況も増えた**
    - (ex)
    - 各画面単位で分担
    - 同じ画面内の画面表示とビジネスロジックで分担
- **自分以外のメンバーがコードを書くことを考えなければならない**
- **メンバー全員がプロジェクトの全貌を理解し、自分のコードの影響範囲を把握し続けることは不可能**
    - (ex)
    - コードに差し込まれた分岐の意図が書いた人にしか分からない
    - 変更の影響範囲を調査するためにプロジェクト全体を検索し、触ったことないコードを丹念に読む必要がある
- **余計なコミュニケーションコストは、開発の障害になり、人数比でそのコストは増大する**
- **新規メンバーがプロジェクトに入ってきたときにも、迅速にキャッチアップできるようにしておきたい**
- 複数人が同じファイルを同時に編集するとコンフリクトやバグの元になる
- それぞれの作業は影響のないようにしたいが、うまく影響範囲が切り分けられないプロジェクトでしばしば衝突事故が発生している
- **各メンバーが自分の領域に集中でき、必要に応じて他所を参照したときにも意図が伝わりやすい、分業に適したコードが必要**

## iOSアプリ設計パターン
- **関心の分離：すべてのコンピュータシステムは、何らかの問題への関心をもち、その問題に対する解決策を提供**
- **問題は、複雑で、いざ解決しようにもさまざまな関心事が絡み合っている**
    - (ex)
    - レイアウトしたい
    - データを組み立てたい
    - 通信したい
    - デバイス情報を受け取りたい
    - 計算したい
    - 文字を解釈したい
    - 状態を管理したい
- **ソフトウェア摂家の根幹にある考え方**
    - **複雑な問題は、より単純な問題の群として切り分けるべき**
    - **そのためには、システムが部品（モジュール）の集合として構成される必要がある**
    - **それぞれの部品は、狭い関心を持ち、一つの問題だけに対処する**
    - **部品が正しく組み合わさって動けば、総体として大きな問題を解決できる**

- 関心の分離の具体例
    - ex1
        - かつてiOSアプリ開発者は、メモリの確保や解放を自らの手で行わなければならなかった
        - インスタンスを作るたびに、確実に参照カウントのインクリメント処理を行い、使い終わるときにはデクリメントするコードを書いていた。
        - メモリリークが発生したときには、コードを追い、カウントのミスを探していた
        - **自動参照カウント（ARC）にという言語機能によって解決**
    - ex2
        - 何台もの検証機やシミュレータをとっかえひっかえしながらレイアウト崩れを確認していた作業をしていた
        - **AutoLayoutの登場で解決**
    - ex3
        - Appleは、iOSアプリ開発に必要な関心事を、UIKitをはじめとしたさまざまなフレームワークへと分離して提供

- **私たちの開発するアプリは、もはやフレームワークにのっとった上でもまだ複雑すぎる**
- **複雑な問題は、より単純な問題の群として切り分ける**
- **強い味方になるのが、設計パターン**
- **設計パターン：再現性のある問題に対する共通の解決策**

- 開発しているアプリが解決したい問題の形はさまざまだが、長いソフトウェアの歴史の中で、まったく新しい問題はそうない
- 過去の達人たちは、何度も設計を繰り返す中で、さまざまな問題に対処するコードが共通して同じパターンに行き着くことを発見
- 解決策に名前をつけ、再利用しやすいようにカタログ化してきた
    - (ex)
    - GoFのデザインパターン

- **普段よく目にする設計パターンは、GoFのデザインパターンに由来するものが多い**
- **頭に入れておくことで、設計の基礎力に役立つ**
    - (ex)
    - Mediator
    - Observer
    - Singleton

### ドメイン駆動設計
- ドメイン駆動設計（DDD）
- **ドメイン：iOSアプリでいえば、UIの都合や通信、永続化の処理を除いた、アプリが解決したい問題領域の知識そのもの**
- **DDDの思想：ドメイン知識のみをレイヤーとして切り出し、そこでは非技術者とも同じ言語を使い、モデリングを反復的に深化させることが価値の高いシステムを生み出すことにつながる**

### パターンを知るメリット
- パターンを知ることのメリット
    - **問題を定型化してとらえられる**
        - 既知のパターンを通して、問題をとらえ直せば、得体の知れなかった魔物が、慣れ親しんだ理解可能な構造に見えてくる
        - わざわざ解決策を再発見する必要もない

    - **解決策を客観的に比較できる**
        - ある問題をパターンを通して捉えるとき、観点によっては複数のパターンが適用できそうに見えるケースがある
            - それぞれのパターンの強み・弱みも、先人によってすでに検討済み
            - **似ている二つのパターンの差分を確認すれば、どちらの方が適しているか客観的に判断できる**
        - パターンによっては、何らかの前提を持っていることがある
            - 成立するケースが限定されている
            - 利便性のため必要なハックがある
            - **実装の先での注意点についても、すべて先人が地図を残してくれている**
            - **細かなトレードオフを把握した上で、パターンを使い分けることで、手戻りは圧倒的に少なくなる**
            - **脳に蓄えたパターンの数は、そのまま自分が選べる武器の数**
    - **メンバーの共通言語になる**
        - 設計が複雑になると、各部品の関連も一目では理解できなくなる
        - 関連図を書いて、ゼロから設計思想を説明することは大変であり、メンバー間で認識の齟齬が生まれる。
        - **「これはXXパターン」と名前が与えられていれば、誰もが一瞬でその構造をイメージできる**
        - **効率よく深井設計の議論ができることは、開発の生産性に大きく貢献する**

### アーキテクチャも「パターン」
- **「アプリを動かす」という複雑な問題領域は、大まかに複数の層（レイヤー）へと切り分けられる。**
- **そこで活用されるのが、アーキテクチャ**
- **それぞれのアプリが対処しようとしている問題は多種多様**
- **問題の解決策となるアーキテクチャも、全く違う形になる**
- **アーキテクチャはパターンである**
- **異なるように見える各アプリの問題も、単純な問題に切り分けたときには、何らかの先人による解決策が適用できるはず**
- **パターンにはトレードオフがある**
- **各パターンを俯瞰的に知り、その比較検討を行う必要がある**

### まとめ
- **設計はアーキテクチャパターンの剪定だけを指すものではない。**
- **切り分けられたレイヤーの内部では、もっと粒度の細かい型が連携しながら処理を行なっている。**
- **レイヤーが抱え込む複雑な問題は、sらにブレークダウンされる必要がある。**
- **ブレークダウンは、十分に問題が単純になるまで続く。**
- **関心んの分離は、再帰的にてきようされなくてはいけない**
- **アーキテクチャを構成するレイヤーのインタフェースも、結局は型によって成り立っている。**
- システムを分割するレイヤーの捉え方の違い
    - (ex)
    - MVC
    - MVP
    - MVVM
    - Flux
    - Clean Architecture