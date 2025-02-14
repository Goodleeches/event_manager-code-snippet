# event_manager-code-snippet
EventManager는 이벤트를 큐에 저장하고 순차적으로 처리하는 시스템입니다.
이벤트는 CallFunc* 타입으로 관리되며, 함수 포인터를 이용해 다양한 이벤트를 처리할 수 있도록 설계되었습니다.

```cpp

#include <deque>
#include <unordered_set>

class EventManager
{
public:
    void PushEvent(CallFunc* pFunc);
    void PopEvent();
    void PopAllEvent();
    CallFunc* FrontEvent();
    bool IsEventQueueEmpty();
    bool IsWaiting();

private:
    std::deque<CallFunc*> m_eventQueue;
    std::unordered_set<CallFunc*> m_eventLookup; 
};

```
```cpp

void EventManager::PushEvent(CallFunc* pFunc)
{
    if (m_eventLookup.find(pFunc) != m_eventLookup.end())
        return; 

    m_eventLookup.insert(pFunc);
    m_eventQueue.push_back(pFunc);
    pFunc->retain(); 
}

CallFunc* EventManager::FrontEvent()
{
    if (m_eventQueue.empty())
        return nullptr;

    return m_eventQueue.front();
}

void EventManager::PopEvent()
{
    if (!m_eventQueue.empty())
    {
        auto pFunc = m_eventQueue.front();
        pFunc->release(); 
        m_eventLookup.erase(pFunc); 
        m_eventQueue.pop_front();
    }
}

void EventManager::PopAllEvent()
{
    while (!m_eventQueue.empty())
    {
        auto pFunc = m_eventQueue.front();
        pFunc->release();
        m_eventLookup.erase(pFunc);
        m_eventQueue.pop_front();
    }
}

bool EventManager::IsEventQueueEmpty()
{
    return m_eventQueue.empty();
}

bool EventManager::IsWaiting()
{
    return IsEventQueueEmpty() &&
           !g_FXMANAGER->IsFxActionRunning() &&
           !g_GAMEMANAGER->getNumberOfRunningActions() &&
           !g_UIMANAGER->getNumberOfRunningActions() &&
           !g_UIMANAGER->IsOpenPopup() &&
           !g_DIALOGMANAGER->IsDialogOpen();
}
