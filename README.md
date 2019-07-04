# A simple Behavior Tree written in C++ for a personal project/assignment.

### Here's the code ->
```cpp
// BTree.cpp : This code belongs to Prathap Ravichander.

#include<pch.h>
#include<iostream>
#include<list>

using namespace std;

class BT
{
public:
	virtual bool runBT() = 0;
};

class CompositeNode : public BT
{
private:
	list<BT*> _Children;
public:
	const list<BT*>& getChildren() const { return _Children; }
	void addChild(BT* _Child)
	{
		_Children.emplace_back(_Child);
	}
};

class Selector : public CompositeNode
{
public:
	virtual bool runBT() override
	{
		for (BT* _Child : getChildren())
		{
			if (_Child->runBT())
			{
				return true;
			}
		}
		return false;
	}
};

class Sequence : public CompositeNode
{
public:
	virtual bool runBT() override
	{
		for (BT* _Child : getChildren())
		{
			if (!_Child->runBT())
			{
				return false;
			}
		}
		return true;
	}
};

class Sequence2 : public CompositeNode
{
	virtual bool runBT() override
	{
		for (BT* _Child : getChildren())
		{
			if (!_Child->runBT())
			{
				return false;
			}
		}
		return true;
	}
};

struct DetectPlayer
{
	bool PlayerSpotted;
	float DistanceToPlayer;
	float MaxWalkSpeed = 150.0f;
};

class PawnSensingTask : public BT
{
private:
	DetectPlayer* CanSeePlayer;
public:
	PawnSensingTask(DetectPlayer* CanSeePlayer) : CanSeePlayer(CanSeePlayer) {}
	virtual bool runBT() override
	{
		if (CanSeePlayer->PlayerSpotted == true)
		{
			CanSeePlayer->MaxWalkSpeed = 400;
			cout << "The AI can see the player" << endl;
		}
		else
		{
			CanSeePlayer->MaxWalkSpeed = 150;
			cout << "The AI hasn't spotted the player yet." << endl;
		}
		return CanSeePlayer->PlayerSpotted;
	}
};

class MoveToPlayerTask : public BT
{
private:
	DetectPlayer* CanSeePlayer;
	bool IsObstructed;
public:
	MoveToPlayerTask(DetectPlayer* CanSeePlayer) : CanSeePlayer(CanSeePlayer) {}
	virtual bool runBT() override
	{
		if (IsObstructed)
		{
			return false;
		}
		if (CanSeePlayer->DistanceToPlayer > 0)
		{
			cout << "The AI is approaching the player" << endl;
			CanSeePlayer->DistanceToPlayer--;
			if (CanSeePlayer->DistanceToPlayer > 1)
			{
				cout << "The AI is now " << CanSeePlayer->DistanceToPlayer << "meters from the player" << endl;
			}
			else if (CanSeePlayer->DistanceToPlayer == 1)
			{
				cout << "ThevAI started it's med range attack" << endl;
			}
			else
			{
				cout << "The AI now begins it's short range attacks" << endl;
			}
		}
		return true;
	}
};

class DestroyPlayerTask : public BT
{
private:
	DetectPlayer* CanSeePlayer;
public:
	DestroyPlayerTask(DetectPlayer* CanSeePlayer) : CanSeePlayer(CanSeePlayer) {}
	virtual bool runBT() override
	{
		if (CanSeePlayer->DistanceToPlayer > 0)
		{
			cout << "The AI is still far away from the player";
			return false;
		}
		CanSeePlayer->PlayerSpotted = true;
		cout << "The AI attacks and damages the player" << endl;
		return true;
	}
};

class LookForPlayerTask : public BT
{
private:
	DetectPlayer* CanSeePlayer;
public:
	LookForPlayerTask(DetectPlayer* CanSeePlayer) : CanSeePlayer(CanSeePlayer) {}
	virtual bool runBT() override
	{
		if (CanSeePlayer->PlayerSpotted == false)
		{
			cout << "The player got away, the AI can't see the player" << endl;
			CanSeePlayer->MaxWalkSpeed = 150;
			cout << "AI goes back to patrolling" << endl;
		}
		else
		{
			cout << "The player is still in the AI's sight" << endl;
		}
		return CanSeePlayer->PlayerSpotted;
	}
};


int main()
{
	Sequence *root = new Sequence, *sequence1 = new Sequence;
	Selector* selector1 = new Selector;
	Sequence2* sequence2 = new Sequence2;
	DetectPlayer* detectPlayer = new DetectPlayer{ false, 10.0f, 150.0f };
	PawnSensingTask* pawnSensing = new PawnSensingTask(detectPlayer);
	MoveToPlayerTask* movetoPlayer = new MoveToPlayerTask(detectPlayer);
	DestroyPlayerTask* destroyPlayer = new DestroyPlayerTask(detectPlayer);
	LookForPlayerTask* lookforPlayer = new LookForPlayerTask(detectPlayer);

	root->addChild(selector1);
	selector1->addChild(sequence1);
	selector1->addChild(sequence2);

	sequence1->addChild(pawnSensing);
	sequence1->addChild(lookforPlayer);

	sequence2->addChild(movetoPlayer);
	sequence2->addChild(destroyPlayer);

	while (!root->runBT())
	{
		cout << "BT not running" << endl;
	}
	cout << "BT executed successfully" << endl;
	cin.get();
}
