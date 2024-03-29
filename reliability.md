



信頼性

何か問題が生じたとしても正しく動作し続けること。

システムの領域には大きく分けてハードウェアの障害、ソフトウェアの障害、そしてヒューマンエラーという3つの信頼性を脅かす要素があるが、これらは可能な限り対策をするべき。

障害を起こしうるものをフォールト、システムが全体として必要なサービスのユーザーへの提供を止めてしまった場合を指す。

このフォールトの存在を事前に見越して対策を取っているようなシステムのことを耐障害性を持つ（フォールトトレラント、もしくはレジリエントである）と言う。


ハードウェアの障害

ハードディスクのクラッシュ、RAMの欠陥、電力網の停電などは常に起こりうるもの。

例えば、ハードディスクの平均故障時間（MTTF:Mean time to failure）はおよそ10年から50年。
10000台のハードディスクを運用している環境では1日に1台壊れるという概算になる。

どのように対策をするのか？

まず最優先事項として、個々のハードウェアに冗長性を持たせることが挙げられる。
ハードディスクはRAID構成にし、サーバーは電源を二重化してホットスワップ（ここではサーバーの電源をつけたまま電力の供給元を切り替えること）可能にし、その上でバックアップ用の電源としてバッテリーとディーゼル発電機を用意する、といった冗長化を行うことでどこか一部分にフォールトが発生しても対応可能となる。

最近ではデータの量やアプリケーションに求められる処理量が増大するにつれて、大量のマシンを使用するアプリケーションが増加している。
大量のマシンを使用するということはすなわちハードウェアのフォールトが発生する確率も高まる、ということになる上、AWSなどのクラウドプラットフォームは単一のマシンの信頼性よりも柔軟性やエラスティック性を重要視して設計されているため、警告なしに仮想マシンのインスタンスが使用できなくなることが一般的。

※エラスティックとは、負荷の増大を検知して自動的にコンピューティングリソースを追加できるシステムのことで、負荷の予想が難しい場合には便利である一方で、運用時に予想外の出来事が発生するという注意点もある。

それ故に、ハードウェアの冗長化に加えてソフトウェア側でも耐障害性の手法を使うことで、マシン全体が突如失われたとしても耐えられるようなシステムへの移行が進んでいる。
単一のマシンしか用意されていない環境ではマシンを再起動する時にはシステムが使用できなくなるダウンタイムを計画しなければならなくなるが、マシンの障害に耐えられるような設計をしているシステムの場合には各マシンに順番にパッチを当てることでシステム全体のダウンタイムを生じさせることなく運用が可能というメリットがある（これをローリングアップグレードという）。

ソフトウェアのエラー

ハードウェアの障害はそれぞれ相関性が薄く、独立しているものと考えられる。
例えば、あるハードディスクに障害が起こっても、それ以外のディスクに障害が発生する可能性は低い（サーバーラックの温度が極端に高いなどの要因があれば別）と考えられる。

一方で、ソフトウェアのフォールトは相関性を持つ傾向にある。
以下はその具体例。
・特定の入力が与えられた場合にはシステム全体がクラッシュする
・一つのコンポーネントのフォールトが他のコンポーネントのフォールトを引き起こすカスケード障害
・特定の処理を行う際のプロセスが暴走し、ハードウェアのリソースを使い切ってしまう
・依存しているシステムが想定しているレスポンスを返さなくなる
など

これらは通常通りではない状況が重なって表面化する時が来るまで潜伏している事がよくある。
そして通常通り、という状況はいつか通常通りではなくなる。

これらに対する対策にハードウェアの冗長化のような手っ取り早い対策は存在せず、ちょっとした積み重ねが重要となる。
例えば、システムの前提について注意深く考察することや、徹底的なテスト、プロセスの分離、プロセスのクラッシュと再起動の許容、計測、モニタリング、プロダクション環境（本番環境）におけるシステムの挙動分析など。

ヒューマンエラー

人間は間違える、最大限努力したとしても信頼性はない。

対策としては、

・ヒューマンエラーの可能性が最小限となるようにシステムを設計する。
上手く設計されている抽象化、API、管理インターフェースは「正しいこと」を行いやすく、「間違ったこと」を回避しやすくしてくれる。
ここで注意すべきなのは、インターフェースの制限が強すぎると人間はそれを回避しようとするため、この方法のメリットが損なわれてしまうということ。

・ヒューマンエラーが発生する可能性が高い部分障害に繋がりやすいところから分離し、サンドボックス環境に移すこと。

・ユニットテストから結合テスト、マニュアルテストまで全てのレベルで徹底的にテストを行うこと。自動化テストだと通常では滅多に発生しないケースを試すのに便利。

・ヒューマンエラーに起因する障害からのリカバリ案の作成。

・パフォーマンスメトリクスやエラーの発生率などのモニタリングの仕組みをセットアップする。
モニタリングを行うことで警告シグナルを早期に受け取る事ができ、何らかの前提や制約に反していないかをチェックできる。
また、問題が発生した場合に分析するためのメトリクスが必要。

・システムの管理方法とトレーニングの実践。

今日のアプリケーションにおいてはクリティカルでない障害でもユーザーに対する責任が存在する。
信頼性を犠牲にする事で開発コストを下げたり、運用コストを下げたりするケースもあるにはあるが、そうしたケースでは信頼性を犠牲にしていることを強く意識すること。



