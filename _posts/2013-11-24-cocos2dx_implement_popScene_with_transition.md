---
layout: post
title: Cocos2d-x对popScene增加转换特效
category: game
---

在Cocos2d-x(cocos2d-x-2.1.4)中，多个scene切换时，可以借助Transition来支持特效，例如pushScene，但是popScene却没有。

Google之，官方论坛给出了办法。在**CCDirector.h**中添加如下代码

	template<typename  T>
	void popSceneWithTransition(float t) {
		CCAssert(m_pRunningScene != NULL, "running scene should not null");
		m_pobScenesStack->removeLastObject();
		unsigned int c = m_pobScenesStack->count();
		if (c == 0) {
			end();
		}
		else {
			m_bSendCleanupToScene = true;
			m_pNextScene = (CCScene*)m_pobScenesStack->objectAtIndex(c - 1);
			CCScene* trans = T::create(t, m_pNextScene);
			m_pobScenesStack->replaceObjectAtIndex(c-1, (CCObject *)trans);
			m_pNextScene = trans;
		}
	}

---
	class FlipXLeftOver: public CCTransitionFlipX
	{
	public:
		static CCTransitionScene* create(float t, CCScene* s) {
			return CCTransitionFlipX::create(t, s, kCCTransitionOrientationLeftOver);
		}
	};
    CCDirector::sharedDirector()->popSceneWithTransition<FlipXLeftOver>(FlipTimeout);

##参考
1. <http://www.cocos2d-x.org/forums/6/topics/14896>