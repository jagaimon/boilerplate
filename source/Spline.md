# スプライン補間をしたい
とにかく補間をしなければいけないことがあります．特に方式にこだわりもない，そんな時にとりあえず選ばれるオーソドックスな方式がスプライン補間だと思います．スプライン補間で描かれた曲線は，必ず既知のデータ上を通ります．  
今回もとりあえずコピペして簡単に使えるC++のコードを用意しました．もちろん動作保証はないです．数式とか細かいことは他のサイトを当たってください.  

基本的な使い方については，まず既知の等間隔のy軸方向のデータをvector型で用意します．次に，そのデータを引数にしてInitParameter関数を呼びます．
計算結果は，0～データ数の間を引数としてCalc関数を呼ぶことで取得することができます．  
以下のサンプルプログラムは0.01間隔でxが0～6の間の結果を取得することで，元データに対して100倍の細かさで補間データを取得する例になります．

```cpp
CSpline calc;
vector<double> sy;//spline用
float skip = 0.01f;//skip幅
sy.clear();

//等間隔のｙデータをセット
sy.push_back(1);//データを入力
sy.push_back(6);//データを入力
sy.push_back(4);//データを入力
sy.push_back(7);//データを入力
sy.push_back(0);//データを入力
sy.push_back(3);//データを入力

calc.InitParameter(sy); //データをセット
for (float i = 0; i < sy.size(); i += skip)
{
	cout << calc.Calc(i) << endl;
}
```

使い方は以上で，以下が実際のコードになります．間違いもあるかと思いますが適当に使ってください．  

header file  
```cpp
#include <vector>

using namespace std;
class CSpline
{
public:
	CSpline();
	~CSpline();

	//データセット関数
	void InitParameter(const vector<double> &y);

	//演算関数
	double Calc(float t);

private:
	vector<double> a_;
	vector<double> b_;
	vector<double> c_;
	vector<double> d_;
	vector<double> w_;
};
```

cpp file  
```cpp
#include "CSpline.h"

CSpline::CSpline()
{
}

CSpline::~CSpline()
{
}

//CSplineデータセット関数
void CSpline::InitParameter(const vector<double> &y) {
	int ndata = (int)y.size() - 1;

	for (int i = 0; i <= ndata; i++)
	{
		a_.push_back(y[i]);
	}

	for (int i = 0; i <= ndata; i++)
	{
		if (i == 0)
		{
			c_.push_back(0.0);
		}
		else if (i == ndata)
		{
			c_.push_back(0.0);
		}
		else
		{
			c_.push_back(3.0*(a_[i - 1] - 2.0*a_[i] + a_[i + 1]));
		}
	}

	for (int i = 0; i < ndata; i++)
	{
		if (i == 0)
		{
			w_.push_back(0.0);
		}
		else
		{
			double tmp = 4.0 - w_[i - 1];
			c_[i] = (c_[i] - c_[i - 1]) / tmp;
			w_.push_back(1.0 / tmp);
		}
	}

	for (int i = (ndata - 1); i > 0; i--)
	{
		c_[i] = c_[i] - c_[i + 1] * w_[i];
	}

	for (int i = 0; i <= ndata; i++) {
		if (i == ndata)
		{
			d_.push_back(0.0);
			b_.push_back(0.0);
		}
		else
		{
			d_.push_back((c_[i + 1] - c_[i]) / 3.0);
			b_.push_back(a_[i + 1] - a_[i] - c_[i] - d_[i]);
		}
	}
}

//CSpline演算関数
double CSpline::Calc(float t)
{
	int j = int(floor(t));
	if (j < 0) {
		j = 0;
	}
	else if (j >= (int)a_.size()) {
		j = ((int)a_.size() - 1);
	}

	double dt = t - j;
	double result = a_[j] + (b_[j] + (c_[j] + d_[j] * dt)*dt)*dt;
	return result;
}
```
