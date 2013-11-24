---
layout: post
title: Cocos2d-x CCTableView的两个bug
category: game
---

##隐藏后再显示无法处理事件

在CCTableView的事件中，隐藏CCTableView，由于此时ccTouchEnded尚未被调用。当隐藏后执行到这里时，被下面的判断给拦截。导致收尾工作没执行。

解决办法是去掉下面两个判断，只需要保留ccTouchBegan的判断。

	void CCTableView::ccTouchEnded(CCTouch *pTouch, CCEvent *pEvent)
	{
		if (!this->isVisible()) {
			return;
		}
	void CCScrollView::ccTouchEnded(CCTouch* touch, CCEvent* event)
	{
		if (!this->isVisible())
		{
			return;
		}
	
##[动态插入删除元素堆栈](http://blog.csdn.net/langresser_king/article/details/9167577)

	void CCTableView::insertCellAtIndex(unsigned  int idx, float insertDelayTime)
	{
		if (idx == CC_INVALID_INDEX)
		{
			return;
		}

		unsigned int uCountOfItems = m_pDataSource->numberOfCellsInTableView(this);
		if (0 == uCountOfItems || idx > uCountOfItems-1)
		{
			return;
		}

		CCTableViewCell* cell = NULL;
		int newIdx = 0;

		cell = (CCTableViewCell*)m_pCellsUsed->objectWithObjectID(idx);
		m_pIndices->clear();
		if (cell)
		{
			newIdx = m_pCellsUsed->indexOfSortedObject(cell);

			for (int i = 0; i < (int)newIdx; ++i) {
				cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(i);
				m_pIndices->insert(cell->getIdx());
			}

			for (unsigned int i=newIdx; i<m_pCellsUsed->count(); i++)
			{
				cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(i);
				int ni = cell->getIdx()+1;
				this->_setIndexForCell(ni, cell, true);
				m_pIndices->insert(ni);
			}
		}

		// [m_pIndices shiftIndexesStartingAtIndex:idx by:1];
		if (insertDelayTime > 0) {
			m_indexToInsert = idx;
			CCDelayTime* delay = CCDelayTime::create(0.2f);
			CCCallFunc* call = CCCallFunc::create(this, callfunc_selector(CCTableView::delayInsertCell));
			CCSequence* seq = CCSequence::createWithTwoActions(delay, call);
			this->runAction(seq);
		} else {
			//insert a new cell
			cell = m_pDataSource->tableCellAtIndex(this, idx);
			this->_setIndexForCell(idx, cell, false);
			this->_addCellIfNecessary(cell);

			this->_updateCellPositions();
			this->_updateContentSize();
		}
	}
	
	void CCTableView::removeCellAtIndex(unsigned int idx)
	{
		if (idx == CC_INVALID_INDEX)
		{
			return;
		}

		unsigned int uCountOfItems = m_pDataSource->numberOfCellsInTableView(this);
		if (0 == uCountOfItems || idx > uCountOfItems-1)
		{
			return;
		}

		unsigned int newIdx = 0;

		CCTableViewCell* cell = this->cellAtIndex(idx);
		if (!cell)
		{
			return;
		}

		newIdx = m_pCellsUsed->indexOfSortedObject(cell);

		//remove first
		this->_moveCellOutOfSight(cell);

	   
		this->_updateCellPositions();
		// [m_pIndices shiftIndexesStartingAtIndex:idx+1 by:-1];

		// 重新修改indices的值
		m_pIndices->clear();
		for (int i = 0; i < (int)newIdx; ++i) {
			cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(i);
			m_pIndices->insert(cell->getIdx());
		}

		for (int i=(int)m_pCellsUsed->count()-1; i >= (int)newIdx; --i)
		{
			cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(i);
			int ni = cell->getIdx()-1;
			this->_setIndexForCell(ni, cell, true);
			m_pIndices->insert(ni);
		}

		// 补充最后一个元素
		if (m_pCellsUsed->count() > 0) {
			cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(m_pCellsUsed->count() - 1);
			int index = cell->getIdx() + 1;
			if (index < (int)m_pDataSource->numberOfCellsInTableView(this)) {
				// 超出显示范围，更新
				this->updateCellAtIndex(index);

				// 更新完毕，重新取最后一个cell
				cell = (CCTableViewCell*)m_pCellsUsed->objectAtIndex(m_pCellsUsed->count() - 1);
				CCPoint dst = cell->getPosition();
				cell->setPositionX(dst.x + m_vCellsPositions[index] - m_vCellsPositions[index - 1]);
				CCMoveTo* moveTo = CCMoveTo::create(0.2f, dst);
				cell->runAction(moveTo);
			}
		}
	}


##参考
1. <http://blog.csdn.net/langresser_king/article/details/9167577>
1. <http://stackoverflow.com/questions/17272569/cocos2dx-cctableview-issue>