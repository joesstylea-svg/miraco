# MIRACO 開発方針

## 進捗メモ（セッション終了時に更新）

- **最終更新:** 2026-06-06（UIブラッシュアップセッション）
- **現在フェーズ:** Phase 3 進行中（CMS化）
- **完了済み（Phase 1-2）:** デザイン基盤 / UX（バリデーション・トースト・戻る・チャット・エンプティステート）
- **完了済み（Phase 3）:** メニュー編集・削除 / ブロックエディタ（見出し・文章・チェックリスト・画像・お客様の声review型）/ 画像アップロード（Canvas圧縮）/ 動的プレースホルダー / 依頼詳細モーダル / 売上自動集計（納品済みメニュー料金合算）/ メニュー別カスタム申込みフォーム（テキスト/ラジオ/チェック・選択肢画像対応）/ ステップ式申込みウィザード（カスタム→基本→その他→確認）
- **完了済み（CMS化）:** siteContent モデル（localStorage `miraco_site`）/ ホーム編集（見出し*強調*・リード・ヒーローカード・画像）/ ギャラリー画面（実績・お客様の声をLP型ブロックで編集）/ ボトムナビ「相談する」→「ギャラリー」に変更 / ロゴクリックでホーム遷移
- **データ保存（2026-06-05 IndexedDB移行）:** localStorageの5MB上限で画像保存がquota超過したため IndexedDB(`miraco_db`/store`kv`)に移行。キー menus/orders/site。起動時 boot() が IDB→無ければ旧localStorage(miraco_menus等)から自動移行。IDB不可環境はlocalStorageにフォールバック。save()は同期呼び出しでOK（内部で非同期書込・throwしない）
- **ブロックtype:** heading / text / checklist / image / imagegrid（content=配列・2列グリッド）/ steps（1行1ステップ・番号付き）/ faq（空行区切り,1行目Q残りA）/ review（1行目=名前,2行目以降=コメント）
- **プレビュー機能（2026-06-05・ポップアップ式に改修）:** ホーム/ギャラリー/メニュー編集に「👁プレビューで確認」。openPreview(applyFn,screenId,title)＝siteContentスナップショット→編集値一時適用→renderSite→対象#screenのinnerHTMLをクローン→siteContent復元→#previewModalにHTML表示（.preview-bodyは*にpointer-events:noneで閲覧専用）。保存されない。メニューはeditingMenu内容で一時tmpメニューをmenusに差し込みrenderDetail→#detail.innerHTMLをクローン→元に戻す。見出し大中小はrenderBlockEditorのheadingにもセレクタ(data-bsize)追加（ギャラリー・メニュー詳細でも使用可、blocksToHTML/renderHomeBlockがlp-h-${size}描画）。
- **設定追加（2026-06-06）:** 管理設定に①運営者パスワード変更(#adminPassInput→siteContent.adminPass)②業種ワード(termService=サービスの呼び方/termRequest=申込の呼び方)。renderSiteで反映：ボトムナビ#navMenuLabel・メニュー見出し#menuPageTitle・ドロワー#drawerMenuLink=「{sw}一覧」、詳細ボタン#detailApplyBtn=「この{sw}を{rw}する」。占い→鑑定/教室→講座等に言い換え可。
- **Supabase接続（2026-06-06 第一歩）:** GitHub repo=joesstylea-svg/miraco（index.html,GitHub Pages公開）。Supabaseプロジェクト=kjnxptncrfaoklkkhtyp（東京リージョン）。テーブル`app_data(key text PK,value jsonb)`にmenus/orders/siteをJSONで保存（単一共有ストア＝1事業者モデル）。index.html内SB_URL/SB_KEY(anon,公開OK)＋fetchでsbGetAll/sbSet。save()はIDB＋Supabase併用、boot()はSupabase優先→IDB/localStorageフォールバック。RLSは現状全許可(public access policy)＝MVP暫定、後で認証導入予定。⚠️DB passwordは絶対共有しない。
- **近リアルタイム同期（2026-06-06）:** startSync()＝12秒ごとにsbGetAllでクラウド確認、orders/menus/site差分があれば反映＆再描画（チャット中はrenderChatで新着反映）。入力中(activeElementがINPUT/TEXTAREA/SELECT)・編集画面(adminHome/Gallery/Edit/Payment/Reviews/apply)は再描画スキップでタイプ妨げない。boot末尾で起動。これで相手の更新が自動で画面に出る＝複数端末で実用的に。注：単一blobモデルのため同時書込はlast-write-winsの弱点あり（将来テーブル分割で解消）。index.htmlも更新済（212717バイト）。
- **QR・アイコン・PWA（2026-06-06）:** icon.svg（ネイビー角丸＋オレンジ✦＋MIRACO）・manifest.json・headにicon/apple-touch-icon(icon.png未作成,要追加)/manifest/theme-color/apple-mobile-web-app追加。共有モーダル#shareModal（QR画像=api.qrserver.com・URL・共有navigator.share・コピー・ホーム画面追加ガイドiPhone/Android）。data-shareでドロワー「📱このアプリを共有/QR」＋管理設定「QRコード・共有」から開く。openShare()。GitHubにicon.svg/manifest.json/index.htmlをアップ必要。iOS用にicon.png(180×180)を後で追加推奨（チャットンで作成）。
- **業種ワード制限:** ヒアリング〜納品の裏側ステータスは固定（カスタム不要と判断）。表の呼び方(termService/termRequest)のみ可変。
- **納品リンク（2026-06-05）:** 容量対策として納品はGoogleドライブ等の共有リンク推奨。order.deliveryLinks[{label,url}]。管理の依頼モーダル納品セクションに「①共有リンクで納品（label+URL,#dlinkAddBtn,削除data-dlinkdel）」＋「②画像直接アップ(deliveryFiles)」。リンク追加でも制作中→確認中。お客様の注文詳細にリンク(.deliv-link,別タブで開く)＋画像DLを表示。base64をアプリに溜めずに大容量・大ファイル納品可、無料。
- **ログイン＆管理保護（2026-06-05）:** お客様ログイン（デモ）＝currentUser(メール,localStorage'miraco_user')。マイページはcurrentUserのo.emailで絞込（未ログインは#loginBox表示）。受注時(submitOrder)に自動ログイン。マジックリンク＝?u=email でログイン（loginLinkBtnでURLコピー,本番はメール送信想定）。ログアウト#logoutBtn。管理画面はドロワーから削除→ドロワー最下部の目立たない歯車#adminGearBtn→prompt(siteContent.adminPass,既定'miraco')でnav('admin')、adminAuthedでセッション中は再入力不要。メニュー詳細の固定バッジ「人気No.1」→menu.badge（編集#eBadge,空で非表示）に動的化。
- **ブランド名一括＆カラーテーマ（2026-06-05）:** ①固定の「MIRACO」表記をsiteContent.brandName/brandSubで一括反映（ヘッダー・ドロワー#drawerBrandName/Sub・ヒーローmock#homeHeroMock・透かし#homeHeroWm・詳細mock#detailHeroMock）。ヒーローバッジはsiteContent.homeHeroBadge（ホーム編集で編集可）。英語キャッチsiteContent.homeEn（空で非表示）。renderSiteで全反映。②カラーテーマTHEMES[5種:ネイビー×オレンジ/モスグリーン×テラコッタ/プラム×ローズ/チャコール×からし/ティール×コーラル]。siteContent.theme(0-4)。applyTheme()がdocument.documentElement.styleで--accent/--accent2/--accent2-deep/--accent-lightを上書き。renderSite冒頭＆boot＆nav('admin')で適用。管理設定に#themePicker（renderThemePicker,data-theme,スウォッチ表示）。
- **バグ修正（2026-06-05）:** renderSiteのギャラリーお客様の声で使うpubRの宣言がhomeBlocksリファクタ時に消えていた→renderSiteが例外で途中停止し、openPreview(renderSite→renderMenus順)のホーム/ギャラリープレビューが無反応に。const pubR=publishedReviews()を復活して解消。
- **ホーム編集の統一＆ロゴ＆フォームプレビュー（2026-06-05）:** ①ホーム下部エディタを共通renderBlockEditor(editingHomeBlocks,'#homeBlocksEditor')に統一（独自renderHomeBlocksEditor廃止）。BLOCK_LABELSにmenus/reviews/worries追加、renderBlockEditorでmenus/reviewsは見出しinput・worriesは案内文。追加はdata-hbadd、移動/削除/編集は共通blockCtx。②メニュー編集の「詳細ページコンテンツ」直後にも👁プレビュー(data-preview-menu)。previewMenu()でメニュー詳細＋formPreviewHTML(editingFormFields)を#previewModalに表示（申込フォームも確認可）。③ロゴ：siteContent.brandName/brandSub/logoImage。ヘッダー#brandLogo(画像)or #brandStar+#brandName/#brandSub(テキスト)をrenderSiteで出し分け。ホーム編集に「ロゴ・ブランド名」セクション（テキスト＋ロゴ画像アップ,高さ34px自動）。見出し大中小はrenderBlockEditorのheadingにdata-bsizeセレクタ。
- **ホーム下部ビルダー化（2026-06-05）:** ヒーロー(タイトル/リード/カード/画像)は固定、下部を#homeBody＝siteContent.homeBlocks駆動に。ブロックtype=heading(size:lg/md/sm 大中小)/text/image/menus(メニューグリッド,content=見出し)/reviews(公開レビュー,content=見出し)/worries(homeWorries使用)。renderHomeBlock()で描画。管理ホーム編集に#homeBlocksEditor（renderHomeBlocksEditor,data-hbadd/hbmv/hbdel/hbsize/hbcontent/hbimg）で並び替え自由。見出しサイズはblocksToHTMLのheadingにもlp-h-${size}適用（全エディタ共通）。renderMenusは#homeCardsを触らない（homeBodyで描画）。worries/reviewsの旧固定セクションHTML削除。
- **モーダル挙動修正（2026-06-05）:** #myOrderModal/#orderModalは背景タップで閉じない（×か「閉じる」のみ）。チャットから詳細へ戻る際はnav('mypage')してからopenMyOrderし、下地をマイページに固定（旧:下地がchatのままで閉じるとチャットに戻るループが発生していた）。階層＝一覧→詳細モーダル→チャット、戻る/閉じるで1段ずつ。ボトムナビ「マイページ」がいつでも一覧へのショートカット。
- **確認フロー＆チャット戻る（2026-06-05）:** お客様の「確認しました」ボタンは撤去（何に対してか不明瞭＋LINE完結勢はマイページ開かないため）。確認は通常のチャット/LINEのやり取りで行い、管理者が定型文で確認依頼→返事を見て「納品済み」にする管理者主導フロー。チャットの戻る(#chatBack)はchatFrom('detail'/'pay'/'mypage')で開いた元へ戻す（注文詳細から開いたら詳細モーダルへ、支払いから開いたら支払いへ）。openChat(no,from)。
- **ダッシュボード整理（2026-06-05）:** 統計とやることの「新着」重複を解消。上段3カード=新着(sNew)/対応中(sDoing=見積/制作/確認)/未入金(sUnpaid)＋独立の売上バナー(.sales-card,支払い済み合計)。すべてタップでフィルター。todoPanelは廃止。確認中=お客様確認待ちの段階→お客様の注文詳細に「✓確認しました」ボタン(data-confirmok,チャットに自動送信)。返事は基本チャット/LINEメッセージ経由。
- **管理機能フル拡充（2026-06-05）:** ①依頼一覧をカード化(.aorder:名前/番号/ステータス/メニュー・受注日・納期/支払バッジ/担当/⚠️納期超過isOverdue/未読)。②担当者(o.assignee 外注先)＋管理メモ(o.adminNote お客様非表示)を依頼モーダルで入力即保存。③チャット定型文(CANNED配列→.canned-chipで返信欄挿入data-canned)。④CSV書き出し(exportCSV,#csvBtn,BOM付UTF-8)。⑤今日のやること#todoPanel(新着/確認待ち/未入金,クリックでフィルター)。⑥メニュー公開停止(menu.hidden,編集の#eHidden,顧客側renderMenusはpub=非hiddenのみ,管理は非公開タグ)。⑦未払いフィルター(orderFilter==='unpaid')追加。納期超過はo.due(日付)が今日より前&未納品。
- **管理ダッシュボード改善（2026-06-05）:** 統計カードをボタン化（data-statfilter:new/doing/paid）→タップで依頼一覧をその条件にフィルター＆カードハイライト。フィルターチップに「支払い済み」追加（計5種:all/new/doing/paid/done、ofMatchでpaid=o.paid対応）。売上=支払い済み(o.paid)注文のquote.total合計（旧:納品済み基準）。マイページ注文カードのヘッダーを「メニュー名＋受注日／納期目安(orderDue:希望納期or menu.due)」に刷新（受付番号は詳細モーダル内のみ）。支払い状況はカード下のpayRow/✓バッジで一目。履歴も「受注 日付」表示。連絡方法o.contactは申込時固定（後からの設定変更は新規注文のみ反映）。
- **ステータス自動連動（2026-06-05）:** 支払い済みにする＆status∈[ヒアリング中,見積もり中]→自動「制作中」。納品データをアップ＆status===制作中→自動「確認中」。トーストで更新を通知。手動変更も従来通り可。
- **管理画面レイアウト整理（2026-06-05）:** 順序＝統計→依頼管理(フィルターチップ:すべて/新着/対応中/納品済みorderFilter＋アーカイブ切替)→メニュー管理(見出し右に＋追加)→設定アコーディオン<details.admin-settings>（ホーム/ギャラリー/支払い/レビュー編集＋連絡方法＋オーナーLINE＋デモリセット）。日々の作業を上、設定は折りたたみ。リセットは設定内に小さく移動。
- **画面整理（2026-06-05）:** estimate画面・delivery画面を削除（機能は注文詳細モーダルに統合済み）。estOrderNo/estPayBtn/estChatBtn/downloadBtn/reviewBtn/data-estimate も削除。注文詳細モーダル＝注文のすべてが見える場所に一本化。注：deliveryFiles（納品物アップ/DL機能）は別物なので存続。
- **レビュー公開管理（2026-06-05）:** お客様レビューはorder.review{rating,text,name,published:false}で保存（自動公開しない）。管理画面「レビュー管理」(#adminReviews)で公開トグル(data-rpub)/削除(data-rdel)。公開済み(publishedReviews())のみホーム(先頭2)・ギャラリー(#galleryReviews,3件＋もっと見るgalleryReviewsExpanded)に掲載。名前はmaskName()でマスク（頭文字＋さん）。マイページに「納品完了！レビュー書きませんか？」案内バナー(納品済み&未レビュー)。siteDefaults.galleryBlocksから既定reviewブロック削除。
- **マイページ機能拡充（2026-06-05）:** ①進捗ステッパーprogressBar(status)＝5段階(ヒアリング/見積り/制作/確認/納品)を注文詳細モーダル上部に表示。②納品物：管理が依頼モーダルでdeliveryFiles[{name,url}]をアップ(#delivUpFile)→お客様の注文詳細で表示＋DL(data-deliv-dl)。③再注文：履歴詳細フッターの「同じ内容でもう一度」(data-reorder)→reorder()でapplyDraft引き継ぎ申込ウィザード。④新着バナー：マイページ上部#msgBannerにお客様未読合計、タップでチャット。⑤レビュー：納品済みの注文詳細で★(1-5)＋コメント→submitReview()でorder.review保存＆siteContent.galleryBlocksにreviewブロック追加（ギャラリーお客様の声に反映）。
- **チャット画像添付（2026-06-05）:** メッセージに image（base64）対応。お客様=チャットバーに📎(#chatFile)→readFile圧縮→pushUserMsg({image})。管理=依頼モーダル返信に📎(#omReplyFile)→pushAdmin({image})。画像はlightbox対応。chatBubble/om-msgが text/image 両対応。
- **LINE/チャット出し分け統一:** myOrderModalのLINE依頼ボタン＝「LINEメッセージ」。支払い画面の相談ボタンも o.contact==='line' なら「LINEメッセージ」(data-line-note→ownerLineUrl)、それ以外は「チャットメッセージ」。
- **オーナーLINE導線（2026-06-05）:** siteContent.ownerLineUrl（管理画面の連絡方法欄に入力,line/bothモード時表示）。お客様の「LINEメッセージ」(data-line-note)→ownerLineUrlを新タブで開く（未設定はトースト）。推奨はLINE公式アカウントの友だち追加URL(lin.ee〜)。個人LINEはID検索制約等で非推奨。
- **用語統一（2026-06-05）:** 「やり取り/チャットで相談」→「チャットメッセージ」、「LINEで連絡/LINEでやり取り」→「LINEメッセージ」。申込の連絡方法選択肢もLINEメッセージ/チャットメッセージ。
- **連絡方法＆スレッドチャット（2026-06-05）:** siteContent.contactMode（line/app/both）を管理画面セレクタで切替。both=申込時にお客様がLINE/アプリ選択（contactPref）→order.contactに保存。lineモードはLINE欄必須・チャット非表示。appモードはLINE欄なし。完了画面もcontactで出し分け（line=LINE案内表示/app=非表示）。チャットは依頼ごとのスレッド（order.messages[{from,text,time,date,readByUser,readByAdmin}]）。未読管理：unreadCount(o,side)/markRead(o,side)。お客様がチャットを開く→管理者msg既読化、管理者がモーダルを開く→お客様msg既読化。マイページやり取りボタン＝未読数バッジ(.chat-badge)、管理依頼行＝未読バッジ(.order-row-unread)、ヘッダー/ドロワーバッジ＝要対応数（新規 or 未読あり,アーカイブ除く）。
- **LINE ID入力補助:** 申込フォームのLINE欄ラベル「LINE ID」。⚠️「表示名ではなくID」注意書き＋折りたたみ`.line-help`に「表示名❌/ID⭕️」の対比ボックス(.lhc bad/good)と確認手順。bothモード時はidNoteで「分からなければアプリ内チャットを選べばOK」と逃げ道提示。
- **バッジ改善:** 要対応＝（ヒアリング中かつ未開封 o.adminViewed=false）or 未読あり。依頼モーダルを開くとadminViewed=true＋既読化で消える。LINE依頼の詳細にlineLink()で「LINEで開く」(@付き=公式https://line.me/R/ti/p/、それ以外=https://line.me/ti/p/~)＋IDコピーボタン。
- **マイページ刷新（2026-06-05）:** 申込中（active=非アーカイブかつ非納品済み）はカード、注文履歴（history=納品済み or アーカイブ）は#myHistoryに一覧表示。「すべてのメニューを見る」削除。カードレイアウト固定化＝上段[詳細を見る｜チャット/LINEメッセージ]の2等分＋下段に全幅[お支払い ¥X]（or ✓支払い済み）。「詳細を見る」(data-myorder)→お客様用注文詳細モーダル#myOrderModal（openMyOrder：申込情報全項目＋見積り内訳＋フッターにチャット/支払い/閉じる）。data-pay/data-chat時は#myOrderModalを閉じてから遷移。estimate画面は実質未使用（estChatBtn/estPayBtnのみ残）。
- **お支払いフロー刷新:** siteContent.paymentMethods[{name,url,info,note}]を管理画面「お支払い方法を編集」(#adminPayment)で設定。url=決済リンク/info=支払い先情報（口座・PayPay ID等,改行可,コピー可）/note=案内文。お客様#pay画面renderPay()：金額を大きく表示＋各方法（案内文・コピー可info・リンクボタンdata-pay-url）＋やり取り誘導＋あとで支払う。発想＝金額はアプリ表示・支払い先は固定情報でお客様が手動で表示金額を支払う（外部リンクに金額埋込不要）。order.paid（admin が依頼モーダル#orderPaidBtnで支払い済みトグル,paidDate記録）。支払い済みはボタン無効・カードに「✓支払い済み」。旧finishPay/自動delivery遷移は廃止。依頼詳細モーダルに「アーカイブに移動/戻す」ボタン。管理依頼一覧は通常=非アーカイブ、#archiveToggleで切替（showArchived）。マイページ・見積りもアーカイブ除外。一覧は新しい順(reverse)。
- **自動見積り（2026-06-05）:** フォーム選択肢（radio/checkbox）に料金を設定可（option.price）。管理の選択肢行に「+¥」入力。申込ウィザードで選択肢に+¥表示＋リアルタイム見積りバー(#quoteBar)。基本料金=メニュー価格をpriceNum()で数値化＋選択オプション加算=請求金額。確認画面・依頼詳細モーダル・見積り画面に内訳表示。calcQuote()=DOM依存/calcQuoteFrom(order)=保存データから再計算。注文にquote{base,addons,total}保存。例：「データでお渡し+¥10,000」
- **ホーム編集で可能:** 見出し*強調* / リード / ヒーローカード文・画像 / 「困った」セクション見出し・ラベル・お悩みリスト
- **自動表示:** ホームにギャラリーのreview先頭2件＋「実績をもっと見る」ボタン / 管理画面の新規依頼数を全画面ヘッダー＆ドロワーにバッジ表示
- **ライトボックス:** LP内画像（image/imagegrid）タップで全画面拡大（#lightbox）
- **配色（2026-06-05 オレンジ主役に再調整）:** 白背景＋オレンジ主役＋ネイビー差し色。--bg #FFFFFF / --ink #1A1A1A / --muted #6E6E6E / --line #E4E4E6 / --accent #235180(ネイビー＝サブ) / --accent2 #FFA62E(明るいオレンジ) / --accent2-deep #F0820F(濃いオレンジ＝CTA・価格・選択状態の白文字用) / --accent-light #FFF1E0(暖色淡) / --paper #F4F5F6。オレンジ適用箇所：primaryボタン・セクション線(.s-line)・進捗バー・ナビアクティブ・ラジオ選択・価格・ヒーローバッジ・★・通知バッジ・ブランド✦。ホームヒーローは暖色グラデ(#FFF6EC→#FFCD9E)。理由：依頼者がベージュ/ブラウンを視認しにくく、オレンジを前面希望
- **申込みウィザード:** 戻る矢印を全ステップ常時表示＋「次へ/確認へ」下にも戻るボタン(#applyBack2、step1は「メニューに戻る」)。希望納期=日付フィールドはタップ全体でカレンダー（showPicker＋calendar-picker-indicatorを全面透明化）、min=今日+最短納期日数(parseLeadDays(menu.due))で納期可能日以降のみ選択可
- **数字フォント（2026-06-05）:** Poppins（Google Fonts読込）を --num-font として全数字に適用（価格・受付番号・見積金額・統計値・ステップ番号・注文番号・管理価格）。Georgia は MIRACOロゴ/モック/見出し/「Q」など文字専用に限定
- **色の最終調整:** 数字系＋primaryボタン＝ネイビー(--accent #235180)。ヒーロー強調文字(h1.page-hero em)＋ステップ円(.step-circle.on/.lp-step-num)＝オレンジ(--accent2-deep)。s-line/進捗/ナビアクティブ/ラジオ選択/バッジ/★/✦＝オレンジ継続
- **ベースファイル変更（2026-06-06）:** `preview (2).html` → `index.html`（GitHub Pages公開用に引っ越し）。CLAUDE.mdのベースファイル記載も要更新。
- **フォントサイズ調整（2026-06-06）:** スマホ可読性向上のため全体的に+1px。body: 14→15px / 本文・説明文: 13→14px / 補助テキスト(.muted): 12→13px / カード説明: 11→12px / 管理入力欄: 13→15px(iOSズーム防止)。意図的に小さいもの(toast/小ボタン/FAQのA/バッジ類)は維持。
- **UIブラッシュアップ完了（2026-06-06）:** フォントサイズ底上げ(body15px・本文14px) / メニュー仕様チップ＆チェックリスト編集機能追加 / プレビューsafe-area対応 / CTAボタン文字編集可能化(homeCtaText) / 日付フォーマット統一(ymdJP/ymdSlash) / カラーテーマ刷新(テラコッタ/インディゴ/フォレスト/バーガンディ) / QRコード公開URL対応(siteContent.publicUrl) / 「気になるものを選んでください」削除 / 管理カードメタ行折り返し防止
- **次にやること:** Phase 4（Supabase・ログイン・本決済・公開）。マイページのチャット導線改善は保留中

## 1. このプロジェクトについて

MIRACOは、ユーワード活動を始めた人の「少し困った」を受け止めるための、スマホファーストの受付アプリです。

名刺づくり、アイキャッチ作成、ホームページメニュー設定、出店1回サポート、ユーワードスタートパック、その他相談などを、スマホからわかりやすく申し込める入口として設計します。

MIRACOは単なるデザイン受注アプリではありません。

「ユーワードに入ったけど、何をしたらいいかわからない」  
「誰に聞けばいいかわからない」  
「ちょっとだけ手伝ってほしい」

そういう人が、安心して相談・依頼できる場所を目指します。

---

## 2. 現在のベースファイル

- `preview (2).html`

このファイルには以下の画面と簡易機能が入っています。

- ホーム / メニュー一覧 / メニュー詳細
- 申込みフォーム / 確認 / 完了
- マイページ / チャット風画面 / お見積り / 支払い方法選択 / 納品ページ
- 管理画面 / メニュー追加
- localStorageによる簡易保存

ゼロから作り直さず、現在の画面遷移・保存処理・データ構造をなるべく活かすこと。

---

## 3. 最優先方針

1. スマホで見たときのアプリ感
2. デザインの世界観
3. 画面遷移の安定
4. 申込み導線のわかりやすさ
5. 管理しやすい構造

Supabase、ログイン、本決済、画像アップロード、通知などの本格機能は後回し。  
まずは1ファイルHTMLで動く「触れる試作品」として完成度を上げる。

**合言葉：機能は後。まず見た目を固定。ゼロから作らない。今のHTMLを育てる。**

---

## 4. デザイン方針

- スマホアプリらしい
- 上品・余白が美しい・雑誌っぽい
- デザイン事務所っぽい洗練感、でも堅苦しくない
- やさしく相談しやすい
- ベージュ、オフホワイト、黒、くすみピンク系
- SaaSテンプレート感を出さない・事務的にしすぎない

「困った人が相談しやすいのに、見た目は洗練されているアプリ」というバランスが重要。

---

## 5. カラールール

```css
--bg: #FAFAF8;
--paper: #F5F2EC;
--ink: #202020;
--muted: #85827D;
--line: #E7E1D8;
--accent: #B8896F;
--accent-dark: #111111;
--soft: #FFFAF4;
/* オプション */
--pink: #D7827C;
--gold: #C9A47E;
--cream: #FAF8F5;
--brown: #7E746B;
```

派手な色は使わない。

---

## 6. フォント方針

- 日本語：`Noto Sans JP`, `Hiragino Sans`, `Zen Kaku Gothic New`
- 英数字・ロゴ補助：`Georgia`, `Inter`

ロゴや大見出しは上品な雰囲気でOK。本文は読みやすさ優先。

---

## 7. UIルール

**余白**
- 情報を詰め込みすぎない
- 上下左右の余白を広めに取る
- 1画面に説明を入れすぎない

**カード**
- 角丸：16〜24px
- 背景：白または薄いベージュ
- 影：ごく薄く / 枠線：薄く上品に

**ボタン**
- メインCTA：黒または濃いブラウン
- サブボタン：白背景＋薄い枠線
- スマホで押しやすい高さ・丸みを持たせる

**入力欄**
- 高さをしっかり取る / 角丸 / 余白広め
- スマホで入力しやすく

---

## 8. 画面構成

### ユーザー側

1. **ホーム** — 世界観・メインコピー・メニュー導線・小さな困った例
2. **メニュー一覧** — 6メニューをわかりやすく選べるリスト
3. **メニュー詳細** — LP風。説明・料金・納期・おすすめ理由・申込みボタン
4. **申込みフォーム** — お名前・メール・LINE・メニュー・相談内容・納期・支払い
5. **確認画面** — 送信前に内容確認
6. **完了画面** — 完了メッセージ・受付番号・マイページ導線
7. **マイページ** — 依頼状況・ステータス・チャット導線・見積り導線
8. **チャット風画面** — デモ動作でOK
9. **お見積り画面** — 料金確認
10. **支払い方法選択** — ユーワードポイント / PayPay / 銀行振込（本決済は後回し）
11. **納品ページ** — 現段階はダミー表示でOK

### 管理者側

1. **管理画面トップ** — 新規依頼数・対応中件数・売上・依頼一覧・メニュー管理
2. **メニュー追加** — 追加したメニューがメニュー一覧に反映されること

---

## 9. 壊してはいけないもの

- `data-nav` による画面遷移
- `data-detail` によるメニュー詳細表示
- 申込みフォームから確認画面への遷移
- 申込み完了後の受付番号表示
- 申込み内容のlocalStorage保存
- 管理画面への依頼反映
- 管理画面からのメニュー追加・一覧反映
- チャット風メッセージ送信
- 支払い完了デモ
- デモデータリセット

---

## 10. 開発フェーズ

| Phase | 内容 | 状態 |
|-------|------|------|
| 1 | デザイン調整（ホーム・メニュー・詳細・フォーム・余白統一） | 🔄 進行中 |
| 2 | 画面遷移・UX調整（戻る導線・マイページ流れ） | ⬜ 未着手 |
| 3 | 管理画面整理（依頼一覧・ステータス・LP編集土台） | ⬜ 未着手 |
| 4 | 本実装（Supabase・ログイン・画像UP・決済・公開） | ⬜ 後回し |

---

## 11. 作業時のルール

**やってよいこと**
- CSS整理・見た目ブラッシュアップ・余白調整
- カード・フォント・ボタン配置調整
- HTML構造の軽い整理・コメント追加

**やってはいけないこと**
- ゼロから全面作り直し
- 画面遷移を壊す
- localStorage保存を削除
- 既存画面を勝手に減らす
- いきなりSupabase・ログイン・決済を入れる
- デザインを一般的なSaaS風に変える

---

## 12. Claude Code への基本指示テンプレート

```
このプロジェクトはMIRACOというスマホファーストの受付アプリです。

`preview (2).html` をベースにしてください。
ゼロから作り直さず、既存の画面遷移・localStorage保存・管理画面・メニュー追加機能を壊さないようにしてください。

目指すデザインは、上品・余白多め・ベージュ・オフホワイト・黒・くすみピンクを基調にした、
スタイリッシュだけど相談しやすいスマホアプリUIです。

まずは機能追加ではなく、UIデザインのブラッシュアップを優先します。
```
