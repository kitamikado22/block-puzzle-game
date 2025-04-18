
/// "Tetris Color" ///

#include "DxLib.h"

// 定数定義
#define STD_MINO_SIZE		4		// 実際に扱うテトリミノのデータサイズ、I_テトリミノの大きさ
#define SPC_MINO_SIZE		3		// S,Z,J,L,T_テトリミノの大きさ
#define S_WIDTH				12		// ステージの幅
#define S_HEIGHT			23		// ステージの高さ
#define DRAW_S_WIDTH		10		// ステージの描画範囲
#define DRAW_S_HEIGHT		20		// ステージの描画範囲
#define MINO_NUM			7		// テトリミノの種類の数
#define NEXT_NUM			5		// ネクストの数
#define X_STAGE_POS			324		// ステージのデータでの左上の座標 x
#define Y_STAGE_POS			108		// ステージのデータでの左上の座標 y
#define X_STAGE_BASE_POS	360		// ステージの左上の座標 x
#define Y_STAGE_BASE_POS	180		// ステージの左上の座標 y
#define X_START_POS			468		// テトリミノの初期座標 x
#define Y_START_POS			144		// テトリミノの初期座標 y
#define X_NEXT_POS			761		// ネクストの共通座標 x
#define Y_NEXT_POS_ONE		195		// ネクスト１の座標 y
#define Y_NEXT_POS_TWO		275		// ネクスト２の座標 y
#define Y_NEXT_POS_THREE	363		// ネクスト３の座標 y
#define Y_NEXT_POS_FORE		451		// ネクスト４の座標 y
#define Y_NEXT_POS_FIVE		539		// ネクスト５の座標 y
#define X_HOLD_POS			253		// ホールドの座標 x
#define Y_HOLD_POS			219		// ホールドの座標 y
#define X_STRING_POS		1332	// 文字列を表示する座標 x
#define Y_TIME_POS			413		// 時間情報の座標 y
#define Y_LINES_POS			305		// ライン情報の座標 y
#define Y_SCORE_POS			521		// スコア情報の座標 y
#define Y_HIGH_SCORE_POS	665		// ハイスコア情報の座標 y
#define BLOCK_PIXEL_SIZE	36		// ブロックのピクセルでの大きさ
#define ADD_STAGE_PIXEL		4		// 21行目のチラ見せの大きさ
#define FALL_TIMING_NUM		60		// テトリミノが自動的に落ちる速度
#define ANIMATION_TIME		5		// アニメーションの描画タイミング
#define LAND_INTERVAL		20		// テトリミノが固定されてからの時間
#define DELETE_INTERVAL		60		// ラインを消してから止まる時間
#define INPUT_TIME			6		// プレイヤーの操作できる速度
#define SOFTDROP_TIME		2		// ソフトドロップ速度
#define ACTIVE_TIME			48		// 遊び時間
#define ACTIVE_NUM			15		// リセットできる回数制限
#define RIGHT				0		// 右
#define LEFT				1		// 左
#define MAX_DELETE_NUM		4		// テトリス
#define CLEAR_ANIME_NUM		9		// ラインを消したアニメーション枚数
#define X_CLEAR_ANIME		308		// ラインを消すエフェクトアニメーションの描画座標 x
#define Y_CLEAR_ANIME_SHIFT	152		// 消すラインのざひょうから合わせるためのずれた座標 y


/* 列挙体 */
// テトリミノの識別番号登録
typedef enum TETRIMINO {
	Space,		// 空白		0
	I_mino,		// 水色		1
	O_mino,		// 黄色		2
	S_mino,		// 緑色		3
	Z_mino,		// 赤色		4
	J_mino,		// 青色		5
	L_mino,		// オレンジ	6
	T_mino,		// 紫色		7
	Wall,		// 壁		8
}Tetrimino;

// 回転パターン
typedef enum ROTATE {
	a,	// 0
	b,	// 1
	c,	// 2
	d,	// 3
}Rotate;


/* グローバル変数 */
// テトリミノのデータ
Tetrimino block_I[STD_MINO_SIZE][STD_MINO_SIZE * 4] = {	// I_テトリミノ
	{0,0,0,0,0,0,1,0,0,0,0,0,0,1,0,0},
	{1,1,1,1,0,0,1,0,0,0,0,0,0,1,0,0},
	{0,0,0,0,0,0,1,0,1,1,1,1,0,1,0,0},
	{0,0,0,0,0,0,1,0,0,0,0,0,0,1,0,0}
};

Tetrimino block_O[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// O_テトリミノ
	{0,2,2,0,2,2,0,2,2,0,2,2},
	{0,2,2,0,2,2,0,2,2,0,2,2},
	{0,0,0,0,0,0,0,0,0,0,0,0}
};

Tetrimino block_S[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// S_テトリミノ
	{0,3,3,0,3,0,0,0,0,3,0,0},
	{3,3,0,0,3,3,0,3,3,3,3,0},
	{0,0,0,0,0,3,3,3,0,0,3,0}
};

Tetrimino block_Z[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// Z_テトリミノ
	{4,4,0,0,0,4,0,0,0,0,4,0},
	{0,4,4,0,4,4,4,4,0,4,4,0},
	{0,0,0,0,4,0,0,4,4,4,0,0}
};

Tetrimino block_J[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// J_テトリミノ
	{5,0,0,0,5,5,0,0,0,0,5,0},
	{5,5,5,0,5,0,5,5,5,0,5,0},
	{0,0,0,0,5,0,0,0,5,5,5,0}
};

Tetrimino block_L[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// L_テトリミノ
	{0,0,6,0,6,0,0,0,0,6,6,0},
	{6,6,6,0,6,0,6,6,6,0,6,0},
	{0,0,0,0,6,6,6,0,0,0,6,0}
};

Tetrimino block_T[SPC_MINO_SIZE][SPC_MINO_SIZE * 4] = {	// T_テトリミノ
	{0,7,0,0,7,0,0,0,0,0,7,0},
	{7,7,7,0,7,7,7,7,7,7,7,0},
	{0,0,0,0,7,0,0,7,0,0,7,0}
};

// ハンドル
int field_handle;				// フィールドのデータハンドル
int stage_back_handle;			// ステージの後ろのデータハンドル
int blocks_handle[MINO_NUM];	// ブロックのデータハンドル
int ghosts_handle[MINO_NUM];	// ゴーストのデータハンドル
int tetrimino_handle[MINO_NUM];	// テトリミノのデータハンドル
int game_over_handle;			// ゲームオーバー
int game_clear_handle;			// ゲームクリア時
int start_reddy;				// スタート時のレディ
int start_go;					// スタート時のレディ
int clear_effect_handle[9];		// ラインを消した時のアニメーション
int font_handle;				// フォントのデータハンドル
int font_color;					// フォントカラー
int best_time[3] = {0};			// ベストタイム


/* クラス */
// 実際に操作するテトリミノのクラス
class cls_Block {
	public:
		// 変数
		Tetrimino mino[STD_MINO_SIZE][STD_MINO_SIZE];	// 実際に扱うテトリミノのデータ
		Tetrimino current;					// 現在扱っているテトリミノの識別番号
		int x;								// x座標
		int y;								// y座標
		int next_complete_pos;				// ネクストがすでに読み込んだ位置座標
		Tetrimino next[NEXT_NUM];			// 公開されるネクスト
		Tetrimino nextbox[2][MINO_NUM];		// ネクストを保管する箱
		Tetrimino hold;						// ホールド
		bool hold_flag;						// ホールドを使ったか使っていないか
		Rotate pattern;						// 回転パターン
		
		// メンバ関数
		cls_Block();				// コンストラクタ
		void create_nextbox();		// ネクストボックスを作る　予備
		void change_nextbox();		// 予備のネクストボックスを第一のネクストボックスに読み込む
		int update_next();			// ネクストボックスから読み込んでネクストを更新
		void update_current();		// ネクストから現在の識別番号を更新
		void load_tetrimino();		// テトリミノをネクストから読み込む
		void set_tetrimino();		// ブロックが着地してから行う処理
		void fall_tetrimino();		// テトリミノの y 座標を下げる
		void move_tetrimino(int);	// テトリミノの x 座標を変える
		void pos_init();			// テトリミノの座標を初期化
		int hold_tetrimino();		// ホールド機能
};

// ステージ上での処理をするクラス
class cls_Stage {
	public:
		Tetrimino stage[S_HEIGHT][S_WIDTH];		// ステージにあるテトリミノのデータ
		int delete_lines_pos[MAX_DELETE_NUM];	// 一度に消すラインの座標を格納
		int delete_lines_num;					// 一度に消すラインの個数
		
		cls_Stage();						// コンストラクタ
		void check_delete_line();			// 消すことのできる行がないか走査
		void delete_lines();				// 指定の行を消す
		void pack_lines();					// 消した分だけ詰める
};

// テトリスのゲームで行う処理をすべて管理するクラス
class cls_Tetris {
	public:
		// オブジェクトの作成
		cls_Block	obj_block;				// 実際に操作するテトリミノの
		cls_Stage	obj_stage;				// ステージ上での処理をする
		XINPUT_STATE pad_input;				// Xinput
		
		// 変数、フラグ
		int count;						// １フレームごとにカウントアップ　60でリセット
		int animation_num;				// アニメーションの枚数を記録
		bool landing;					// テトリミノが着地したかのフラグ
		bool clearing;					// ラインを消したかどうかのフラグ
		bool game_over_flag;			// ゲームオーバーしているかのフラグ
		bool animation_finish_flag;		// アニメーションの描画終了のフラグ
		bool game_finish_flag;			// ゲームを辞めるかどうか
		bool loop_finish_flag;			// ループ終了
		bool game_start_flag;			// スタート時
		bool game_clear_flag;			// ゲームクリア時
		bool active_time_flag;			// 遊び時間のフラグ
		bool player_control_flag;		// プレイヤーの操作
		int key[256];					// すべてのキー状態を取得する
		int input[32];					// パッドのキー状態を取得する
		int stick[4];					// スティック
		int ghost;						// ゴーストの座標
		char m_second;					// 時間　ms
		char second;					// 時間 秒
		int minute;						// 時間　分
		int score;						// スコア
		int lines;						// ライン
		int active_count;				// 遊び時間を連続で活用した回数
		//int pad_state;					// パッド状態
		
		// メンバ関数
		cls_Tetris();						// コンストラクタ
		void save_best_time();				// ベストタイムを記録
		void print_screen();				// 画面描画
		void start_game();					// スタート時の演出
		void print_game_over();				// ゲームオーバー時の演出
		void print_game_clear();			// ゲームオーバー時の演出
		void set_hold();					// ホールド
		void control();						// プレイヤーの操作
		int control_pad();					// コントローラー操作
		bool check_hit_block(int, int);		// テトリミノが壁またはブロックと重なっているか調べる
		void fixe_tetrimino();				// テトリミノをステージに固定
		void set_ghost();					// ゴースト機能
		void rotate(int);					// 回転処理
		void set_lines();					// テトリミノが固定されてからの処理
		void set_activ_time();				// テトリミノが固定されてからの遊び時間
		bool check_game_over();				// ゲームオーバーしているか確認
		bool check_game_clear();			// ゲームクリアしているか確認
		void process();						// テトリス内部処理
};


/* メンバ関数 */
// コンストラクタ
cls_Block::cls_Block()
{
	// 座標初期設定
	x = X_START_POS;
	y = Y_START_POS;
	
	// ホールド初期化
	hold = Space;
	hold_flag = false;
	
	// 回転パターン初期化
	pattern = a;
	
	// ネクストボックス作成
	for (int i = 0; i < MINO_NUM; i++) {
		nextbox[0][i] = i + 1;
		nextbox[1][i] = i + 1;
	}
	for (int i = 0; i < 2; i++) {
		for (int j = 0; j < MINO_NUM; j++) {
			int a = GetRand(MINO_NUM - 1);
			int b = GetRand(MINO_NUM - 1);
			Tetrimino tmp	= nextbox[i][a];
			nextbox[i][a]	= nextbox[i][b];
			nextbox[i][b]	= tmp;
		}
	}
	
	// ネクストがすでに読み込んだ位置座標を初期化
	next_complete_pos = 0;
	
	// ネクストの作成
	for (int i = 0; i < NEXT_NUM; i++) {
		next[i] = nextbox[0][i];
		next_complete_pos++;	// 座標更新
	}
}

// ネクストボックスを作る 予備
void cls_Block::create_nextbox()
{
	// 順番に数字（カウントアップ）を入れる
	for (int i = 0; i < MINO_NUM; i++) {
		nextbox[1][i] = i + 1;
	}
	
	// ランダムに入れ替える
	for (int i = 0; i < MINO_NUM; i++) {
		int a = GetRand(MINO_NUM - 1);
		int b = GetRand(MINO_NUM - 1);
		Tetrimino tmp	= nextbox[1][a];
		nextbox[1][a]	= nextbox[1][b];
		nextbox[1][b]	= tmp;
	}
}

// 予備のネクストボックスを第一のネクストボックスに読み込む
void cls_Block::change_nextbox()
{
	for (int i = 0; i < MINO_NUM; i++) {
		nextbox[0][i] = nextbox[1][i];
	}
}

// ネクストボックスから読み込んでネクストを更新
int cls_Block::update_next()
{
	// ネクストをずらす
	for (int i = 0; i < NEXT_NUM; i++) {
		next[i] = next[i + 1];
	}
	
	// 最後尾に読み込む
	next[NEXT_NUM - 1] = nextbox[0][next_complete_pos];
	
	// 座標更新
	next_complete_pos++;
	
	// ネクストに予備のネクストボックスのみ読み込んでいる場合
	if (next_complete_pos - MINO_NUM == NEXT_NUM) {
		next_complete_pos = NEXT_NUM;	// 初期化
		return 1;
	}
	return 0;
}

// ネクストから現在の識別番号を更新
void cls_Block::update_current()
{
	// 識別番号の更新
	current = next[0];
}

// テトリミノを読み込む
void cls_Block::load_tetrimino()
{
	// I_テトリミノを読み込む場合
	if (current == I_mino) {
		for (int i = 0; i < STD_MINO_SIZE; i++) {
			for (int j = 0; j < STD_MINO_SIZE; j++) {
				mino[i][j] = block_I[i][j + pattern * STD_MINO_SIZE];
			}
		}
	}
	
	// それ以外を読み込む場合
	else {
		for (int i = 0; i < STD_MINO_SIZE; i++) {
			for (int j = 0; j < STD_MINO_SIZE; j++) {
				mino[i][j] = Space;		// 初期化
			}
		}
		
		switch (current) {
			case 2:		// O_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_O[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
			case 3:		// S_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_S[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
			case 4:		// Z_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_Z[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
			case 5:		// J_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_J[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
			case 6:		// L_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_L[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
			case 7:		// T_テトリミノ
				for (int i = 0; i < SPC_MINO_SIZE; i++) {
					for (int j = 0; j < SPC_MINO_SIZE; j++) {
						mino[i][j] = block_T[i][j + pattern * SPC_MINO_SIZE];
					}
				}
				break;
		}
	}
}

// ブロックが着地してから行う処理
void cls_Block::set_tetrimino()
{
	// 回転パターン初期化
	pattern = a;
	
	// ネクストから現在の識別番号を更新
	update_current();
	
	// テトリミノのロード
	load_tetrimino();
	
	// ネクストの更新
	if (update_next())	// ネクストのロード位置が予備だけになっている場合
	{
		// 予備のネクストボックスを第一のネクストボックスにロード
		change_nextbox();
		
		// 予備のネクストボックスの作成
		create_nextbox();
	}
}

// テトリミノの y 座標を下げる
void cls_Block::fall_tetrimino()
{
	y += BLOCK_PIXEL_SIZE;
}

// テトリミノの x 座標を変える
void cls_Block::move_tetrimino(int dir)
{
	// 右に動かす場合
	if (dir == RIGHT) {
		x += BLOCK_PIXEL_SIZE;
	}
	
	// 左に動かす場合
	else {
		x -= BLOCK_PIXEL_SIZE;
	}
}

// テトリミノの座標を初期化
void cls_Block::pos_init()
{
	// 座標を初期化
	x = X_START_POS;
	y = Y_START_POS;
}

// ホールド機能
int cls_Block::hold_tetrimino()
{	
	// テトリミノを扱っている場合、一度しかホールドは使えない
	if (hold_flag) {
		return 0;
	}
		
	// ホールドする
	hold_flag = true;
	Tetrimino tmp = hold;
	hold = current;
	
	// ホールドに何もない場合
	if (tmp == Space) {
		return 1;
	}
	
	// ホールドから取り出す
	current = tmp;
	
	// テトリミノのロード
	load_tetrimino();
	
	return 0;
}

// コンストラクタ
cls_Stage::cls_Stage()
{
	// ステージの初期化
	for (int i = 0; i < S_HEIGHT; i++) {
		for (int j = 0; j < S_WIDTH; j++) {
			stage[i][j] = Space;
			
			// 壁の作成
			if (j == 0 || j == S_WIDTH - 1 || i == S_HEIGHT - 1) {
				stage[i][j] = Wall;
			}
		}
	}
	
	// 変数初期化
	delete_lines_num = 0;
}

// 消すことのできる行がないか走査
void cls_Stage::check_delete_line()
{
	//int line_flag;
	delete_lines_num = 0;
	
	for (int i = 0; i < DRAW_S_HEIGHT; i++) {
		
		//line_flag = 0;	// 初期化
		
		for (int j = 0; j < DRAW_S_WIDTH; j++) {
			
			// 空白が見つかればその行ではない
			if (stage[i + 2][j + 1] == Space) {
				break;
			}
			
			// DRAW_S_WIDTHの大きさ分加算されればその行はブロックが詰まっている
			//line_flag++;
			// 最後まで空白が見つからなけばその行を消す
			if (j == DRAW_S_WIDTH - 1) {
				// 入っている座標は描画範囲をベースにしていることに注意
				delete_lines_pos[delete_lines_num] = i;
				delete_lines_num++;
				//printfDx("delete_lines_num:%d\n", delete_lines_num);
			}
		}
	}
}

// 指定の行を消す
void cls_Stage::delete_lines()
{
	for (int i = 0; i < delete_lines_num; i++) {
		for (int j = 0; j < DRAW_S_WIDTH; j++) {
			stage[delete_lines_pos[i] + 2][j + 1] = Space;
		}
	}
}

// 消した分だけ詰める
void cls_Stage::pack_lines()
{
	// 下のほうから一行ずつ詰めていく
	for (int i = 0; i < delete_lines_num; i++) {
		// 消す座標を出す
		int pos = delete_lines_pos[delete_lines_num - 1 - i];
		int j = 0;
		int now, next;
		
		// すべてずらしたら終了
		while (true)
		{
			// 詰める場所と一つ上
			now = pos - j + 2;
			next = now - 1;
			
			// 一番上に来たら
			if (next == 0) {
				for (int k = 0; k < DRAW_S_WIDTH; k++) {
					stage[next][k + 1] = Space;		// 初期化
				}
				break;
			}
			
			// 一つ上の行をコピー
			for (int k = 0; k < DRAW_S_WIDTH; k++) {
				stage[now][k + 1] = stage[next][k + 1];
			}
			
			j++;
		}
		
		// すらしたので消す座標もずらす
		for (int k = 0; k < delete_lines_num - i; k++) {
			delete_lines_pos[k]++;
		}
	}
}

// コンストラクタ
cls_Tetris::cls_Tetris()
{	
	// 変数、フラグ初期化
	count = 0;
	animation_num = 0;
	landing = false;
	clearing = false;
	game_over_flag = false;
	animation_finish_flag = false;
	game_finish_flag = false;
	loop_finish_flag = false;
	game_start_flag = true;
	game_clear_flag = false;
	active_time_flag = false;
	player_control_flag = false;
	ghost = 0;
	m_second = 0;
	second = 0;
	minute = 0;
	score = 0;
	lines = 40;
	active_count = 0;
	//pad_state = 0;
	
	// テトリミノの事前登録
	obj_block.set_tetrimino();
	
	// パッドが接続されているか調べる
}

// ベストタイムを記録
void cls_Tetris::save_best_time()
{
	for (int i = 0; i < 3; i++) {
		if (best_time[i] != 0) break;
		if (i == 2) {
			best_time[0] = minute;
			best_time[1] = second;
			best_time[2] = m_second;
		}
	}
	
	if (minute < best_time[0]) {
		best_time[0] = minute;
		if (second < best_time[1]) {
			best_time[1] = second;
			if (m_second < best_time[2]) {
				best_time[2] = m_second;
			}
		}
	}
}

// 画面描画
void cls_Tetris::print_screen()
{
	// ステージの後ろの描画
	DrawGraph(X_STAGE_BASE_POS, Y_STAGE_BASE_POS - ADD_STAGE_PIXEL, stage_back_handle, FALSE);
	
	// テトリミノの描画 着地したりラインを消した場合は描画しない
	if (clearing == false && landing == false) {
		for (int i = 0; i < STD_MINO_SIZE; i++) {
			for (int j = 0; j < STD_MINO_SIZE; j++) {
				
				// 空白は処理を飛ばす
				if (obj_block.mino[i][j] == Space) {
					continue;
				}
				
				// ブロックの識別番号の通りに画像を描画
				int num = obj_block.mino[i][j] - 1;
				int x = obj_block.x + j * BLOCK_PIXEL_SIZE;
				int y = obj_block.y + i * BLOCK_PIXEL_SIZE;
				
				// ゴーストの描画
				DrawGraph(x, y + ghost, ghosts_handle[num], TRUE);
				
				// テトリミノの描画
				DrawGraph(x, y, blocks_handle[num], TRUE);
			}
		}
	}
	
	// ステージの描画
	for (int i = 0; i < DRAW_S_HEIGHT; i++) {
		for (int j = 0; j < DRAW_S_WIDTH; j++) {
			
			// 空白は処理を飛ばす
			if (obj_stage.stage[i + 2][j + 1] == Space) {
				continue;
			}
			
			// ブロックの識別番号の通りに画像を描画
			int num = obj_stage.stage[i + 2][j + 1] - 1;
			int x = X_STAGE_BASE_POS + j * BLOCK_PIXEL_SIZE;
			int y = Y_STAGE_BASE_POS + i * BLOCK_PIXEL_SIZE;
			DrawGraph(x, y, blocks_handle[num], TRUE);
		}
	}
	
	// フィールドの描画
	DrawGraph(0, 0, field_handle, TRUE);
	
	// ラインを消した時のアニメーション
	if (clearing) {
		for (int i = 0; i < obj_stage.delete_lines_num; i++) {
			int y = Y_STAGE_BASE_POS + obj_stage.delete_lines_pos[i] * BLOCK_PIXEL_SIZE - Y_CLEAR_ANIME_SHIFT;
			DrawGraph(X_CLEAR_ANIME, y, clear_effect_handle[CLEAR_ANIME_NUM - animation_num], TRUE);
		}
		
		if (count % ANIMATION_TIME == 0) {
			// アニメーション描画終了
			if (animation_num == 1) {
				animation_finish_flag = true;
			}
			animation_num--;
		}
	}
	
	// ネクストの描画
	DrawGraph(X_NEXT_POS, Y_NEXT_POS_ONE, tetrimino_handle[obj_block.next[0] - 1], TRUE);
	DrawGraph(X_NEXT_POS, Y_NEXT_POS_TWO, tetrimino_handle[obj_block.next[1] - 1], TRUE);
	DrawGraph(X_NEXT_POS, Y_NEXT_POS_THREE, tetrimino_handle[obj_block.next[2] - 1], TRUE);
	DrawGraph(X_NEXT_POS, Y_NEXT_POS_FORE, tetrimino_handle[obj_block.next[3] - 1], TRUE);
	DrawGraph(X_NEXT_POS, Y_NEXT_POS_FIVE, tetrimino_handle[obj_block.next[4] - 1], TRUE);
	
	// ホールドの描画
	if (obj_block.hold > Space) {
		DrawGraph(X_HOLD_POS, Y_HOLD_POS, tetrimino_handle[obj_block.hold - 1], TRUE);
	}
	
	// 情報描画
	DrawFormatStringToHandle(X_STRING_POS, Y_TIME_POS, font_color, font_handle, "%2d:%2d:%2d", minute, second, m_second);
	DrawFormatStringToHandle(X_STRING_POS, Y_LINES_POS, font_color, font_handle, "%8d", lines);
	DrawFormatStringToHandle(X_STRING_POS, Y_SCORE_POS, font_color, font_handle, "%8d", score);
	DrawFormatStringToHandle(X_STRING_POS, Y_HIGH_SCORE_POS, font_color, font_handle, "%2d:%2d:%2d", best_time[0], best_time[1], best_time[2]);
	
	// ゲームスタート
	if (game_start_flag) {
		start_game();
	}
	
	// ゲームオーバー
	if (game_over_flag) {
		print_game_over();
	}
	
	// ゲームクリア
	if (game_clear_flag) {
		print_game_clear();
	}
}

// スタート時の演出
void cls_Tetris::start_game()
{
	if (count <= 60) {
		DrawExtendGraph(X_STAGE_POS+count, Y_STAGE_POS+count, X_STAGE_POS+800-count, Y_STAGE_POS+800-count, start_reddy, TRUE);
	}
	else if (count <= 80){
		int shift = 20 * (count - 60);
		DrawExtendGraph(X_STAGE_POS+390-shift, Y_STAGE_POS+390-shift, X_STAGE_POS+410+shift, Y_STAGE_POS+410+shift, start_go, TRUE);
	}
	else if (count <= 100) {
		DrawGraph(X_STAGE_POS, Y_STAGE_POS, start_go, TRUE);
	}
	else if (count <= 110) {
		int shift = 40 * (count - 100);
		DrawExtendGraph(X_STAGE_POS, Y_STAGE_POS+shift, X_STAGE_POS+800, Y_STAGE_POS+800-shift, start_go, TRUE);
	}
	else {
		count = 0;
		game_start_flag = false;
	}
}

// ゲームオーバー時の演出
void cls_Tetris::print_game_over()
{
	// 等加速度運動
	int y = (count * count / 2) - 500;
	
	if (y > Y_NEXT_POS_TWO) {
		y = Y_NEXT_POS_TWO;
	}
	if (count == 240) {
		game_finish_flag = true;
		game_over_flag = false;
	}
	
	DrawGraph(X_STAGE_POS, y, game_over_handle, TRUE);
	
}

// ゲームクリア時の演出
void cls_Tetris::print_game_clear()
{
	// 等加速度運動
	int y = (count * count / 2) - 500;
	
	if (y > Y_NEXT_POS_TWO) {
		y = Y_NEXT_POS_TWO;
	}
	if (count == 240) {
		game_finish_flag = true;
		game_clear_flag = false;
	}
	
	DrawGraph(X_STAGE_POS, y, game_clear_handle, TRUE);
	
	// ベストタイムを記録
	save_best_time();
}

// ホールドの安全チェック
void cls_Tetris::set_hold()
{
	if (obj_block.hold_tetrimino()) {
		int tmp = obj_block.current;
		obj_block.update_current();
		obj_block.load_tetrimino();
		
		if (check_hit_block(0,0) == false) {
			if (check_hit_block(1,0)) {
				obj_block.x += BLOCK_PIXEL_SIZE;
			}
			else if (check_hit_block(-1,0)) {
				obj_block.x -= BLOCK_PIXEL_SIZE;
			}
			else {
				obj_block.current = tmp;
				obj_block.load_tetrimino();
				return;
			}
		}
		obj_block.set_tetrimino();
		return;
	}
	if (check_hit_block(0,0) == false) {
		if (check_hit_block(1,0)) {
			obj_block.x += BLOCK_PIXEL_SIZE;
		}
		else if (check_hit_block(-1,0)) {
			obj_block.x -= BLOCK_PIXEL_SIZE;
		}
		else {
			obj_block.hold_flag = false;
			obj_block.hold_tetrimino();
		}
	}
}

// プレイヤーの操作
void cls_Tetris::control()
{
	// 処理ができない
	if (landing || clearing || game_over_flag || game_start_flag) {
		return;
	}
	player_control_flag = false;
	
	// パッド入力
	/*
	if (control_pad() == 0) {
		return;
	}*/
	
	// すべてのキー状態を取得する
	char tmpkey[256];
	GetHitKeyStateAll(tmpkey);
	
	// 押し続けると加算され続ける
	for (int i = 0; i < 256; i++) {
		if (tmpkey[i]) {
			key[i]++;
		}
		else key[i] = 0;
	}
	
	// 右に動かす
	if (key[KEY_INPUT_D] % INPUT_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(1, 0) == true) {
			obj_block.move_tetrimino(RIGHT);
		}
		player_control_flag = true;
		return;
	}
	
	// 左に動かす
	if (key[KEY_INPUT_A] % INPUT_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(-1, 0) == true) {
			obj_block.move_tetrimino(LEFT);
		}
		player_control_flag = true;
		return;
	}
	
	// ソフトドロップ
	if (key[KEY_INPUT_S] % SOFTDROP_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(0, 1) == true) {
			obj_block.fall_tetrimino();
		}
		else {
			active_time_flag = true;
		}
		return;
	}
	
	// ハードドロップ
	if (key[KEY_INPUT_W] == 1) {
		count = 0;					// 初期化
		obj_block.y += ghost;		// 座標をゴーストに合わせる
		fixe_tetrimino();			// 固定
		landing = true;
		return;
	}
	
	// ホールド
	if (key[KEY_INPUT_LSHIFT] == 1 || key[KEY_INPUT_RSHIFT] == 1) {
		set_hold();
		return;
	}
	
	// 右回転
	if (key[KEY_INPUT_RIGHT] == 1) {
		rotate(RIGHT);
		player_control_flag = true;
		return;
	}
	
	// 左回転
	if (key[KEY_INPUT_LEFT] == 1) {
		rotate(LEFT);
		player_control_flag = true;
		return;
	}
	
	// esc
	if (key[KEY_INPUT_ESCAPE] == 1) {
		game_finish_flag = true;
		loop_finish_flag = true;
		return;
	}
}

// コントローラー操作
int cls_Tetris::control_pad()
{
	int state = GetJoypadXInputState(DX_INPUT_PAD1, &pad_input);
	//printfDx("state:%d", state);
	//int input[16] = {0};
	
	if (state == -1) return -1;
	
	// 押し続けると加算され続ける
	for (int i = 0; i < 16; i++) {
		if (pad_input.Buttons[i] == 1) {
			input[i]++;
		}
		else input[i] = 0;
	}
	/*
	if (pad_input.ThumbLX > 0) {
		stick[XINPUT_BUTTON_DPAD_RIGHT]++;
	} else stick[XINPUT_BUTTON_DPAD_RIGHT] = 0;
	if (pad_input.ThumbLX < 0) {
		stick[XINPUT_BUTTON_DPAD_LEFT]++;
	} else stick[XINPUT_BUTTON_DPAD_LEFT] = 0;
	if (pad_input.ThumbLY < 0) {
		stick[XINPUT_BUTTON_DPAD_DOWN]++;
	} else stick[XINPUT_BUTTON_DPAD_DOWN] = 0;
	if (pad_input.ThumbLY > 0) {
		stick[XINPUT_BUTTON_DPAD_UP]++;
	} else stick[XINPUT_BUTTON_DPAD_UP] = 0;
	*/
	
	// 右に動かす
	if (input[XINPUT_BUTTON_DPAD_RIGHT] % INPUT_TIME == 1 || stick[XINPUT_BUTTON_DPAD_RIGHT] % INPUT_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(1, 0) == true) {
			obj_block.move_tetrimino(RIGHT);
		}
	}
	//printfDx("input:%d", input[XINPUT_BUTTON_DPAD_RIGHT]);
	
	// 左に動かす
	if (input[XINPUT_BUTTON_DPAD_LEFT] % INPUT_TIME == 1 || stick[XINPUT_BUTTON_DPAD_LEFT] % INPUT_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(-1, 0) == true) {
			obj_block.move_tetrimino(LEFT);
		}
	}
	
	// ソフトドロップ
	if (input[XINPUT_BUTTON_DPAD_DOWN] % SOFTDROP_TIME == 1 || stick[XINPUT_BUTTON_DPAD_DOWN] % SOFTDROP_TIME == 1) {
		
		// 壁やブロックに当たると動かせない
		if (check_hit_block(0, 1) == true) {
			obj_block.fall_tetrimino();
		}
	}
	
	// ハードドロップ
	if (input[XINPUT_BUTTON_DPAD_UP] == 1 || stick[XINPUT_BUTTON_DPAD_UP] == 1) {
		count = 0;					// 初期化
		obj_block.y += ghost;		// 座標をゴーストに合わせる
		fixe_tetrimino();			// 固定
		landing = true;
	}
	
	// ホールド
	if (input[XINPUT_BUTTON_LEFT_SHOULDER] == 1 || input[XINPUT_BUTTON_RIGHT_SHOULDER] == 1) {
		set_hold();
	}
	
	// 右回転
	if (input[XINPUT_BUTTON_B] == 1) {
		rotate(RIGHT);
	}
	
	// 左回転
	if (input[XINPUT_BUTTON_A] == 1) {
		rotate(LEFT);
	}
	
	// esc
	if (input[XINPUT_BUTTON_START] == 1) {
		game_finish_flag = true;
		loop_finish_flag = true;
	}
	
	return state;
}

// テトリミノが壁またはブロックと重なっているか調べる
bool cls_Tetris::check_hit_block(int add_x, int add_y)
{
	// データでの座標に変換
	int x = (obj_block.x - X_STAGE_POS) / BLOCK_PIXEL_SIZE;
	int y = (obj_block.y - Y_STAGE_POS) / BLOCK_PIXEL_SIZE;
	
	for (int i = 0; i < STD_MINO_SIZE; i++) {
		for (int j = 0; j < STD_MINO_SIZE; j++) {
			
			// 空白の部分は調べない
			if (obj_block.mino[i][j] == Space) {
				continue;
			}
			
			// ブロックのある座標に壁またはブロックがないか調べる
			if (obj_stage.stage[y + i + add_y][x + j + add_x] > 0) {
				return false;
			}
		}
	}
	return true;
}

// テトリミノをステージに固定
void cls_Tetris::fixe_tetrimino()
{
	//printfDx("TEST\n");
	// データでの座標に変換
	int x = (obj_block.x - X_STAGE_POS) / BLOCK_PIXEL_SIZE;
	int y = (obj_block.y - Y_STAGE_POS) / BLOCK_PIXEL_SIZE;
	
	for (int i = 0; i < STD_MINO_SIZE; i++) {
		for (int j = 0; j < STD_MINO_SIZE; j++) {
			
			// 空白の部分は調べない
			if (obj_block.mino[i][j] == Space) {
				continue;
			}
			
			// ステージのデータを上書き
			obj_stage.stage[y + i][x + j] = obj_block.mino[i][j];
		}
	}
}

// ゴースト機能
void cls_Tetris::set_ghost()
{
	// 仮想の y 座標初期化
	int virtual_y = 0;
	
	while (true) {
		if (check_hit_block(0, virtual_y) == false) {
			virtual_y--;
			break;
		}
		virtual_y++;
	}
	
	// ピクセルでの座標に変換
	ghost = virtual_y * BLOCK_PIXEL_SIZE;	
}

// 回転処理
void cls_Tetris::rotate(int dir)
{
	// I_テトリミノの場合
	if (obj_block.current == I_mino) {
		// 右回転
		if (dir == RIGHT) {
			obj_block.pattern++;
			if (obj_block.pattern > d) {
				obj_block.pattern = a;
			}
			
			// テトリミノをロード
			obj_block.load_tetrimino();
			
			// スーパーローテーションシステム
			if (check_hit_block(0, 0)) {	// case 0
				return;
			}
			
			switch (obj_block.pattern) {
				case a:		// pattern a
					if (check_hit_block(-2, 0)) {	// case 1
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, 0)) {	// case 2
						obj_block.x += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, 2)) {	// case 3
						obj_block.x += BLOCK_PIXEL_SIZE;
						obj_block.y += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, -1)) {	// case 4
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						obj_block.y -= BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = d;
					obj_block.load_tetrimino();
					break;
				case b:		// pattern b
					if (check_hit_block(-2, 0)) {	// case 1
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, 0)) {	// case 2
						obj_block.x += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, 1)) {	// case 3
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						obj_block.y += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, -2)) {	// case 4
						obj_block.x += BLOCK_PIXEL_SIZE;
						obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = a;
					obj_block.load_tetrimino();
					break;
				case c:		// pattern c
					if (check_hit_block(-1, 0)) {	// case 1
						obj_block.x -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, 0)) {	// case 2
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, -2)) {	// case 3
						obj_block.x -= BLOCK_PIXEL_SIZE;
						obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, 1)) {	// case 4
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						obj_block.y += BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = b;
					obj_block.load_tetrimino();
					break;
				case d:		// pattern d
					if (check_hit_block(2, 0)) {	// case 1
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, 0)) {	// case 2
						obj_block.x -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, -1)) {	// case 3
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						obj_block.y -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, 2)) {	// case 4
						obj_block.x -= BLOCK_PIXEL_SIZE;
						obj_block.y += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = c;
					obj_block.load_tetrimino();
					break;
			}
		}
		
		// 左回転
		else if (dir == LEFT) {
			obj_block.pattern--;
			if (obj_block.pattern < a) {
				obj_block.pattern = d;
			}
			
			// テトリミノをロード
			obj_block.load_tetrimino();
			
			// スーパーローテーションシステム
			if (check_hit_block(0, 0)) {	// case 0
				return;
			}
			
			switch (obj_block.pattern) {
				case a:		// pattern a
					if (check_hit_block(2, 0)) {	// case 1
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, 0)) {	// case 2
						obj_block.x -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, -1)) {	// case 3
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						obj_block.y -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, 2)) {	// case 4
						obj_block.x -= BLOCK_PIXEL_SIZE;
						obj_block.y += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = b;
					obj_block.load_tetrimino();
					break;
				case b:		// pattern b
					if (check_hit_block(1, 0)) {	// case 1
						obj_block.x += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, 0)) {	// case 2
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, 2)) {	// case 3
						obj_block.x += BLOCK_PIXEL_SIZE;
						obj_block.y += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, -1)) {	// case 4
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						obj_block.y -= BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = c;
					obj_block.load_tetrimino();
					break;
				case c:		// pattern c
					if (check_hit_block(1, 0)) {	// case 1
						obj_block.x += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, 0)) {	// case 2
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-2, 1)) {	// case 3
						obj_block.x -= 2 * BLOCK_PIXEL_SIZE;
						obj_block.y += BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(1, -2)) {	// case 4
						obj_block.x += BLOCK_PIXEL_SIZE;
						obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = d;
					obj_block.load_tetrimino();
					break;
				case d:		// pattern d
					if (check_hit_block(-1, 0)) {	// case 1
						obj_block.x -= BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, 0)) {	// case 2
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(-1, -2)) {	// case 3
						obj_block.x -= BLOCK_PIXEL_SIZE;
						obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
						return;
					}
					if (check_hit_block(2, 1)) {	// case 4
						obj_block.x += 2 * BLOCK_PIXEL_SIZE;
						obj_block.y += BLOCK_PIXEL_SIZE;
						return;
					}
					
					// 元に戻す
					obj_block.pattern = a;
					obj_block.load_tetrimino();
					break;
			}
		}
		return;
	}

	// 通常処理
	// 右回転
	if (dir == RIGHT) {
		obj_block.pattern++;
		if (obj_block.pattern > d) {
			obj_block.pattern = a;
		}
		
		// テトリミノをロード
		obj_block.load_tetrimino();
		
		// スーパーローテーションシステム
		if (check_hit_block(0, 0)) {	// case 0
			return;
		}
		
		switch (obj_block.pattern) {
			case a:		// pattern a
				if (check_hit_block(-1, 0)) {	// case 1
					obj_block.x -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, 1)) {	// case 2
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, -2)) {	// case 3
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, -2)) {	// case 4
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = d;
				obj_block.load_tetrimino();
				break;
			case b:		// pattern b
				if (check_hit_block(-1, 0)) {	// case 1
					obj_block.x -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, -1)) {	// case 2
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, 2)) {	// case 3
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, 2)) {	// case 4
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = a;
				obj_block.load_tetrimino();
				break;
			case c:		// pattern c
				if (check_hit_block(-1, 0)) {	// case 1
					obj_block.x -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, 1)) {	// case 2
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, -2)) {	// case 3
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, -2)) {	// case 4
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = b;
				obj_block.load_tetrimino();
				break;
			case d:		// pattern d
				if (check_hit_block(1, 0)) {	// case 1
					obj_block.x += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, -1)) {	// case 2
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, 2)) {	// case 3
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, 2)) {	// case 4
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = c;
				obj_block.load_tetrimino();
				break;
		}
	}
	
	// 左回転
	else if (dir == LEFT) {
		obj_block.pattern--;
		if (obj_block.pattern < a) {
			obj_block.pattern = d;
		}
		
		// テトリミノをロード
		obj_block.load_tetrimino();
		
		// スーパーローテーションシステム
		if (check_hit_block(0, 0)) {	// case 0
			return;
		}
		
		switch (obj_block.pattern) {
			case a:		// pattern a
				if (check_hit_block(1, 0)) {	// case 1
					obj_block.x += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, 1)) {	// case 2
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, -2)) {	// case 3
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, -2)) {	// case 4
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = b;
				obj_block.load_tetrimino();
				break;
			case b:		// pattern b
				if (check_hit_block(-1, 0)) {	// case 1
					obj_block.x -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, -1)) {	// case 2
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, 2)) {	// case 3
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(-1, 2)) {	// case 4
					obj_block.x -= BLOCK_PIXEL_SIZE;
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = c;
				obj_block.load_tetrimino();
				break;
			case c:		// pattern c
				if (check_hit_block(1, 0)) {	// case 1
					obj_block.x += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, 1)) {	// case 2
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, -2)) {	// case 3
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, -2)) {	// case 4
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y -= 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = d;
				obj_block.load_tetrimino();
				break;
			case d:		// pattern d
				if (check_hit_block(1, 0)) {	// case 1
					obj_block.x += BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, -1)) {	// case 2
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y -= BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(0, 2)) {	// case 3
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				if (check_hit_block(1, 2)) {	// case 4
					obj_block.x += BLOCK_PIXEL_SIZE;
					obj_block.y += 2 * BLOCK_PIXEL_SIZE;
					return;
				}
				
				// 元に戻す
				obj_block.pattern = a;
				obj_block.load_tetrimino();
				break;
		}
	}
}

// テトリミノが固定されてからの処理
void cls_Tetris::set_lines()
{
	// 消すことのできる行がないか走査
	obj_stage.check_delete_line();
	
	// 消すラインが見つかった場合
	if (obj_stage.delete_lines_num > 0)
	{
		// 指定の行を消す
		obj_stage.delete_lines();
		
		clearing = true;
		lines -= obj_stage.delete_lines_num;
		animation_num = CLEAR_ANIME_NUM;
	}
}

// テトリミノが固定されてからの遊び時間
void cls_Tetris::set_activ_time()
{
	// 遊び時間である条件か調べる
	if (check_hit_block(0, 1) == true) {
		active_time_flag = false;
		active_count = 0;
		count = 0;
		return;
	}
	
	// 遊び時間を過ぎた場合終了
	// 遊び時間を連続で活用した回数制限に達した場合終了
	if (count > ACTIVE_TIME || active_count > ACTIVE_NUM) {
		active_time_flag = false;
		active_count = 0;
		count = FALL_TIMING_NUM - (40 - lines) - 1;
		return;
	}
	
	// 遊び時間中にプレイヤーの操作が行われた場合
	if (player_control_flag) {
		count = 0;
		active_count++;
	}
}

// ゲームオーバーしているか確認	true:ゲームオーバー  false:問題ない
bool cls_Tetris::check_game_over()
{
	// 新しいテトリミノが出現するときにほかのブロックと重なっていないか調べる
	if (check_hit_block(0, 0) == false) {
		return true;
	}
	else return false;
	
	// 21段目にブロックがおいていないか調べる
	for (int i = 0; i < DRAW_S_WIDTH; i++) {
		if (obj_stage.stage[1][i + 1] == Space) {
			return false;
		}
	}
	return true;
}

// ゲームをクリアしているか確認
bool cls_Tetris::check_game_clear()
{
	if (lines <= 0) return true;
	else			return false;
}

// 内部処理
void cls_Tetris::process()
{
	// フレームごとにカウントアップ
	count++;
	
	// 処理をやらない
	if (game_over_flag || game_finish_flag || game_start_flag || game_clear_flag) {
		return;
	}
	
	// 時間処理
	m_second++;
	if (m_second == 60) {
		m_second = 0;
		second++;
	}
	if (second == 60) {
		second = 0;
		minute++;
	}
	
	// ラインを消したのなら処理を飛ばす
	if (animation_finish_flag)
	{
		// 初期化
		clearing = false;
		animation_finish_flag = false;
		count = 0;
		
		// ブロックを詰める
		obj_stage.pack_lines();
	}
	if (clearing)	return;
	
	// ブロックが着地したら
	if (landing && count == LAND_INTERVAL)
	{
		// 変数処理
		count = 0;
		
		// 消すことのできる行がある場合処理
		set_lines();
		
		// テトリミノの登録
		obj_block.set_tetrimino();
		
		// テトリミノの座標を初期化
		obj_block.pos_init();
		
		// ゲームオーバーしているか確認
		if (check_game_over()) {
			game_over_flag = true;
		}
		
		// ゲームをクリアしているか確認
		if (check_game_clear()) {
			game_clear_flag = true;
		}
		
		// フラグを回収
		landing = false;				// 着地
		obj_block.hold_flag = false;	// ホールド
	}
	
	// 着地してからのインターバル中は残りの処理を飛ばす
	if (landing || clearing)	return;

	// 遊び時間中に確認作業
	if (active_time_flag) {
		set_activ_time();
		return;
	}
	
	// ブロックを自動的に一段落とす
	if (count == FALL_TIMING_NUM - (40 - lines))
	{
		// カウンター初期化
		count = 0;
		
		// 着地するか調べる
		if (check_hit_block(0, 1) == false) {
			fixe_tetrimino();	// 固定
			landing = true;
			return;
		}
		
		// y 座標を下げる
		obj_block.fall_tetrimino();
		
		// 遊び時間が発生するか調べる
		if (check_hit_block(0, 1) == false) {
			active_time_flag = true;
		}
	}
	
	// ゴースト
	set_ghost();
}


// データのロード
void load_data()
{
	// 画像データの読み込み
	field_handle = LoadGraph("image/screen.png");
	stage_back_handle = LoadGraph("image/stage_back.png");
	game_over_handle = LoadGraph("image/game_over.png");
	game_clear_handle = LoadGraph("image/game_clear.png");
	start_reddy = LoadGraph("image/reddy.png");
	start_go = LoadGraph("image/go.png");
	LoadDivGraph("image/blocks.png", MINO_NUM, MINO_NUM, 1, 36, 36, blocks_handle);
	LoadDivGraph("image/ghosts.png", MINO_NUM, MINO_NUM, 1, 36, 36, ghosts_handle);
	LoadDivGraph("image/tetrimino.png", MINO_NUM, MINO_NUM, 1, 92, 63, tetrimino_handle);
	LoadDivGraph("image/clear_line.png", 9, 3, 3, 460, 336, clear_effect_handle);
	
	// フォントの作成
	font_handle = CreateFontToHandle(NULL, 50, 5);
	
	// 色の取得
	font_color = GetColor(241, 241, 241);
}



/* メイン関数 */
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
	SetGraphMode(1920, 1080, 32);	// 画面モードの設定
	ChangeWindowMode(TRUE);			// ウィンドウモードに設定
	if (DxLib_Init() == -1) {		// DXライブラリ初期化処理
		return -1;					// エラーが起きたら直ちに終了
	}
	SetDrawScreen(DX_SCREEN_BACK);	// 描画先を裏画面に設定
	
	// データのロード
	load_data();
	
	while (true) {
		// オブジェクトの生成
		cls_Tetris obj_tetris;
		
		// メインループ
		while (ProcessMessage() == 0 && obj_tetris.game_finish_flag == false)
		{
			// プレイヤーの操作
			obj_tetris.control();
			
			// 内部処理全般
			obj_tetris.process();
			
			// 裏画面を描画
			obj_tetris.print_screen();
			
			// 裏画面を表画面に反映
			ScreenFlip();
		}
		
		// ループ終了
		if (obj_tetris.loop_finish_flag)	break;
	}
	
	// DXライブラリ終了処理
	DxLib_End();
	return 0;
}