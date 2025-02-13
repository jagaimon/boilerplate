# ベジェ曲線を描きたい
人生においてベジェ曲線を引かなければならないことがあるかと思います．  
そんな時のために，コピペして何となく動くであろうC++のソースコードを用意しました．理屈や原理・数式は他のサイトを参照してください．  
このコードは入力した制御点に対して指定の刻みでベジェ曲線の座標を出力するというものです．  
ベジェ曲線は両端の制御点以外は通らないため，データ自体を補間するような用途より，何となく近似した滑らかな曲線を描くというようなデザイン的な用途に向くと思われます．  
早速，今回のコードの使い方ですが，まずfSkipに出力の刻みを0～1の値で指定します．下の例では制御点のXが1～6のため，0.01と入力することで，入力範囲は600個になります．  
次にxとyに制御点をpushしていきます．その後，CalcBezierCurve()で演算を実行するとVecResultXとVecResultYに結果が出力されます．

```cpp
CBezierCurve calc;//定義
calc.fSkip = 0.01f;//0～1の間で出力間隔を指定(入力範囲を何刻みにするか）

calc.x.clear();
calc.y.clear();

calc.x.push_back(1);//制御点を入力
calc.y.push_back(1);//制御点を入力

calc.x.push_back(2);//制御点を入力
calc.y.push_back(6);//制御点を入力

calc.x.push_back(3);//制御点を入力
calc.y.push_back(4);//制御点を入力

calc.x.push_back(4);//制御点を入力
calc.y.push_back(7);//制御点を入力

calc.x.push_back(5);//制御点を入力
calc.y.push_back(0);//制御点を入力

calc.x.push_back(6);//制御点を入力
calc.y.push_back(10);//制御点を入力

calc.CalcBezierCurve();//演算実行

for (int i = 0; i < (int)calc.VecResultX.size(); i++)//結果を出力
{
	cout << calc.VecResultX[i] << " " << calc.VecResultY[i] << endl;
}
```

使い方は以上で，以下が実際のコードになります．間違いもあるかと思いますが適当に使ってください．  

header file  
```cpp
using namespace std;

class CBezierCurve
{
public:
	CBezierCurve();
	~CBezierCurve();

	//ベジェ曲線計算関数
	void CalcBezierCurve();

	//バーンスタイン基底関数
	double Bernstein(int, int, float);

	//二項係数を計算する関数
	double Binomial(int, int);

	//階乗計算を行う関数
	double Factorial(int);


	//べき乗計算
	double ipow(double, int);

	// 測定データ
	vector<double> x; // 入力データX 
	vector<double> y; // 入力データY 
	vector<double> VecResultX; //出力データX
	vector<double> VecResultY; //出力データY 
	float fSkip;	 // 結果出力時の出力間隔 
};
```

cpp file  
```cpp
#include "CBezierCurve.h"

CBezierCurve::CBezierCurve()
{
	fSkip = 0.0f;
	x.clear();
	y.clear();
}


CBezierCurve::~CBezierCurve()
{
}


//ベジェ曲線計算関数
void CBezierCurve::CalcBezierCurve()
{
	double resultX = 0.0;
	double resultY = 0.0;
	int n = (int)x.size();

	VecResultX.clear();
	VecResultY.clear();
	double dSkipOne;
	dSkipOne = 1.0f / ((double)n / fSkip);
	for (double t = 0.0f; t <= 1.0f; t += dSkipOne)
	{
		resultX = 0.0;
		resultY = 0.0;
		for (int i = 0; i < n; i++)
		{
			resultX += x[i] * Bernstein(n - 1, i, (float)t); 
			resultY += y[i] * Bernstein(n - 1, i, (float)t);
		}
		VecResultX.push_back(resultX);
		VecResultY.push_back(resultY);
	}
}

//べき乗計算を行う関数
double CBezierCurve::ipow(double p, int n)
{
	int k;
	double s = 1;

	for (k = 1; k <= n; k++)
	{
		s *= p;
	}
	return s;
}

//バーンスタイン基底関数
double CBezierCurve::Bernstein(int n, int i, float t)
{
	return Binomial(n, i) * ipow(t, i) * ipow(1 - t, n - i);
}


//二項係数を計算する関数
double CBezierCurve::Binomial(int n, int k)
{
	return Factorial(n) / (Factorial(k) * Factorial(n - k));
}


//階乗計算を行う関数
double CBezierCurve::Factorial(int a)
{
	double result = 1.0f;
	for (int i = 2; i <= a; i++)
	{
		result *= i;
	}

	return result;
}
```