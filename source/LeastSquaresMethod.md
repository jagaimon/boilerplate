# 理屈はいいから最小二乗法を使いたい
とにかく数式とか理屈はいいからコピペしてコードを使いたいということがありますよね.  
そういうときのためのC++のコードです．  
このコードは最小二乗法を用いて入力した制御点に対して指定の刻みで補間を行うというものです．  
もちろん動作保証はないですが，コピペして何となく動くと思います．数式とか細かいことは他のサイトを当たってください.  
  
基本的な使い方については，まず0~1の間でfSkipに刻みを設定します．これは，入力したXの範囲に対してそれをどれくらいの刻みで解を求めるかという値になります．例えば下の例ではskipに0.01を指定するとxが1～6までなので500個に分割されて解が出力されます．  
次にMに次数をセットします．次数は高ければ高いほど精度が上がりますが，予期せぬ暴れ方をする可能性が高くなります．いくつがいいかということを理由付けて決めるのは難しいです．入力するデータに対する出力の傾向を見て設定してください．  
次にxとyに既知の制御点をpushしていきます．  
その後，CalcLeastSquaresMethod()を実行して演算を行います．演算結果はVecResultXとVecResultYにそれぞれ入ります．  

```cpp
CLeastSquaresMethod calc;//定義
calc.fSkip = 0.01f;//0～1の間で補間間隔を指定(入力範囲を何刻みにするか）
calc.M = 3;  //次数を指定

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

calc.CalcLeastSquaresMethod();//演算実行

for (int i = 0; i < (int)calc.VecResultX.size(); i++)//結果を出力
{
	cout << calc.VecResultX[i] << endl;
	cout << calc.VecResultY[i] << endl;
}

```

使い方は以上で，以下が実際のコードになります．間違いもあるかと思いますが適当に使ってください．  

header file  
```cpp
#include <vector>

using namespace std;

class CLeastSquaresMethod
{
public:
	CLeastSquaresMethod();
	~CLeastSquaresMethod();

	void CalcLeastSquaresMethod();

	double ipow(double, int);       // べき乗計算	
	// 測定データ
	vector<double> x;
	vector<double> y;
	vector<double> VecResultX; //出力データX
	vector<double> VecResultY; //出力データY 
	float fSkip;
	int M;    // 予測曲線の次数
};
```
  
cpp file
```cpp
#include "CLeastSquaresMethod.h"

CLeastSquaresMethod::CLeastSquaresMethod()
{
	fSkip = 0.0f;
	x.clear();
	y.clear();
}


CLeastSquaresMethod::~CLeastSquaresMethod()
{
}

void CLeastSquaresMethod::CalcLeastSquaresMethod()
{
	int i, j, k, m2, mp1, mp2;
	double *a, aik, pivot, *w, w1, w2, w3;
	double p,  px;

	VecResultX.clear();
	VecResultY.clear();
	int m = M;
	int n = y.size();//data数
	double *c = new double[M + 1];

	if (m >= n || m < 1)
	{
		return; //error
	}
	mp1 = m + 1;
	mp2 = m + 2;
	m2 = 2 * m;
	a = (double *)malloc(mp1 * mp2 * sizeof(double));
	if (a == NULL)
	{
		return; //error
	}
	w = (double *)malloc(mp1 * 2 * sizeof(double));
	if (w == NULL)
	{
		free(a);
		return; //error
	}
	for (i = 0; i < m2; i++)
	{
		w1 = 0.;
		for (j = 0; j < n; j++)
		{
			w2 = w3 = x[j];
			for (k = 0; k < i; k++) {
				w2 *= w3;
			}
			w1 += w2;
		}
		w[i] = w1;
	}
	a[0] = n;
	for (i = 0; i < mp1; i++)
	{
		for (j = 0; j < mp1; j++)
		{
			if (i || j) {
				a[i * mp2 + j] = w[i + j - 1];
			}
		}
	}

	w1 = 0.;
	for (i = 0; i < n; i++)
	{
		w1 += y[i];
	}
	a[mp1] = w1;
	for (i = 0; i < m; i++)
	{
		w1 = 0.;
		for (j = 0; j < n; j++)
		{
			w2 = w3 = x[j];
			for (k = 0; k < i; k++) {
				w2 *= w3;
			}
			w1 += y[j] * w2;
		}
		a[mp2 * (i + 1) + mp1] = w1;
	}

	for (k = 0; k < mp1; k++)
	{
		pivot = a[mp2 * k + k];
		a[mp2 * k + k] = 1.0;
		for (j = k + 1; j < mp2; j++) {
			a[mp2 * k + j] /= pivot;
		}
		for (i = 0; i < mp1; i++)
		{
			if (i != k)
			{
				aik = a[mp2 * i + k];
				for (j = k; j < mp2; j++)
				{
					a[mp2 * i + j] -= aik * a[mp2 * k + j];
				}
			}
		}
	}
	for (i = 0; i < mp1; i++)
	{
		c[i] = a[mp2 * i + mp1];
	}
	free(w);
	free(a);

	for (px = x[0]; px <= x[x.size() - 1]; px += fSkip) {
		p = 0;
		for (k = 0; k <= M; k++)
		{
			p += c[k] * ipow(px, k);
		}
		VecResultX.push_back(px);
		VecResultY.push_back(p);
	}
	free(c);
	return;
}


//べき乗計算
double CLeastSquaresMethod::ipow(double p, int n)
{
	int k;
	double s = 1;

	for (k = 1; k <= n; k++)
	{
		s *= p;
	}

	return s;
}
```