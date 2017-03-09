---
layout: post
title: Warp Speed Effect Tutorial
image: https://img.youtube.com/vi/pB9UAb-b168/maxresdefault.jpg
---

You can't have a good spaceship game without warp speed, and this is mine.

<iframe width="560" height="315" src="https://www.youtube.com/embed/pB9UAb-b168" frameborder="0" allowfullscreen></iframe>

This is a diagram of all the major components and their relationships. All the children of *Effects* have nearly identical structures, so it's not as complicated as it looks.

<img src="/images/chart-warp-structure.svg" alt="" />
<!--![Warp Architecture](images/chart-warp-structure.svg)-->

## Manager

At the top level we have a facade that provides a simple interface for interacting with all the other components.

<img src="/images/inspector-warp-manager.png" alt="" />

`Engage()` starts all the components and sets the event handlers. The end of warp is triggered from the `WarpPlayer` component's OnComplete event trigger. From there we shake the camera, tell the `WarpEffects` to exit, and propogate the `OnComplete` event.

The `OnUpdate` handler is there to stop the stars particle systems. This is so that the stars end slightly before the player reaches the destination. This was just a visual preference.

    public class WarpManager : MonoBehaviour {

        public WarpUI warpUI;
        public WarpEffects effects;
        public WarpPlayer player;
        public WarpCamera warpCamera;
        public ThrusterGroup thrusters;

        public Transform cameraTarget;
        public float speed;
        public GoEaseType easeType;

        private Action<AbstractGoTween> _onComplete;

        public Vector3 Destination { get; set; }

        public void Warp()
        {
            warpUI.ChargeComplete += Engage;
            warpUI.Begin();
        }

        private void Engage()
        {
            effects.EnterWarp();
            thrusters.SetMaxPower();
            player.SetOnCompleteHandler(delegate (AbstractGoTween t)
            {
                //Go.to(warpCamera.transform, 0.5f, new GoTweenConfig().shake(new Vector3(1, 1, 1), GoShakeType.Position, 2));
                GameManager.instance.CameraController.ShakeCamera(0.4f, 10, 0.3f);
                effects.ExitWarp();
                _onComplete.Invoke(t);
            });
            player.SetOnUpdateHandler(delegate (AbstractGoTween t)
            {
                if (t.totalDuration - t.totalElapsedTime < 0.8f) effects.EndStars();
            });
            player.Warp(Destination, speed, easeType);
            Debug.Log("Warp called");

            var camOffset = CalculatePositionOffset(cameraTarget.position, player.transform.position);
            warpCamera.Warp(Destination + camOffset, speed, easeType);
        }

        public void SetOnCompleteHandler(Action<AbstractGoTween> handler)
        {
            _onComplete = handler;
        }

        private Vector3 CalculatePositionOffset(Vector3 offsetFromTarget, Vector3 target)
        {
            var x = offsetFromTarget.x - target.x;
            var y = offsetFromTarget.y - target.y;
            var z = offsetFromTarget.z - target.z;
            return new Vector3(x, y, z);
        }
    }

## UI

This controls the UI elements: the minigame shamelessly stolen from Gears of War, and the countdown text.

The implementation details of the minigame and the countdown are beyond the scope of this tutorial, but what's important is the interfaces they provide to this component. The interface of the `WarpUI` simplifies the interfaces of those 2 components even further (hey, another facade!). `Begin()` starts the countdown and minigame, and the `ChargeComplete` event triggers when the countdown is finished.

    public class WarpUI : MonoBehaviour
    {
        public BarTimingMiniGame miniGame;
        public UICountdown countdown;

        public delegate void ReadyDelegate();
        public event ReadyDelegate ChargeComplete;

        public void Begin()
        {
            BeginCountdown();
            BeginMinigame();
        }

        private void BeginCountdown()
        {
            countdown.Finished += Countdown_Finished;
            countdown.Play();
        }

        private void BeginMinigame()
        {
            miniGame.ResultReady += MiniGame_ResultReady;
            miniGame.StartMiniGame();
        }

        private void OnChargeComplete()
        {
            if (ChargeComplete != null) ChargeComplete();
        }

        private void Countdown_Finished()
        {
            OnChargeComplete();
        }

        private void MiniGame_ResultReady(Enums.MiniGameResult result)
        {
            switch (result)
            {
                case Enums.MiniGameResult.perfect:
                    countdown.TimeLeft = 0.01f;
                    break;
                case Enums.MiniGameResult.good:
                    countdown.TimeLeft = 3f;
                    break;
                default:
                    break;
            }
        }
    }

## Effects

This controls the image effects and other aesthetic changes.

<img src="/images/inspector-warp-effects.png" alt="" />

Each of these effects has its own script to simplify this class. As it is, this class is another facade.

    public class WarpEffects : MonoBehaviour {

        [Header("Radial Blur")]
        public WarpRadialBlur radialBlur;
        public float radialBlurStrength = 1f;
        [Header("Vignette")]
        public WarpVignette vignette;
        public float vignetteStrength = 1.4f;
        [Header("Stars")]
        public WarpStarsGroup warpStars;
        [Header("Field of View")]
        public WarpFov warpFov;
        public float fOVZoomInValue = 50f;
        [Header("Other Details")]
        public float startDuration = 1f;
        public float endDuration = 1f;
        public GoEaseType easeType = GoEaseType.Linear;

        void Start ()
        {
            Invoke("Warp", 2f);
        }

        public void EnterWarp()
        {
            BeginRadialBlur();
            BeginVignette();
            ShrinkFieldOfView();
            BeginStars();
        }

        public void ExitWarp()
        {
            GrowFieldOfView();
            EndVignette();
            EndRadialBlur();
        }

        private void ShrinkFieldOfView()
        {
            warpFov.Begin(startDuration, fOVZoomInValue, easeType);
        }

        private void GrowFieldOfView()
        {
            warpFov.End(endDuration, easeType);
        }

        private void BeginRadialBlur()
        {
            radialBlur.Begin(startDuration, radialBlurStrength, easeType);
        }

        private void EndRadialBlur()
        {
            radialBlur.End(endDuration, easeType);
        }

        private void BeginVignette()
        {
            vignette.Begin(startDuration, vignetteStrength, easeType);
        }

        private void EndVignette()
        {
            vignette.End(endDuration, easeType);
        }

        private void BeginStars()
        {
            warpStars.Begin(startDuration, easeType);
        }

        public void EndStars()
        {
            warpStars.End(endDuration * 0.1f, easeType);
        }
    }

### Radial Blur, Vignette, and Field of View

All of these effects have nearly identical scripts. The interface has just `Begin()` and `End()` for entering and exiting warp.

[Screengrab of WarpEffects inspector]

Note that the `duration` and `strength` parameters are passed in rather than being specified in these components. This was done so that all the variables controlling the effects would be provided from one place in the inspector: the `WarpEffects` component. This lets you change the duration in one place, and change all the effects at once.

    [RequireComponent(typeof(RadialBlur))]
    public class WarpRadialBlur : MonoBehaviour {

        private RadialBlur _radialBlur;
        private GoTween _beginTween;

        void Start()
        {
            _radialBlur = GetComponent<RadialBlur>();
        }

        public void Begin(float duration, float strength, GoEaseType easeType = GoEaseType.Linear)
        {
            _beginTween = Go.to(_radialBlur, duration, new GoTweenConfig().floatProp("BlurStrength", strength).setEaseType(easeType));
        }

        public void End(float duration, GoEaseType easeType = GoEaseType.Linear)
        {
            if (_beginTween.state == GoTweenState.Running)
            {
                _beginTween.pause();
                _beginTween.destroy();
            }

            Go.to(_radialBlur, duration, new GoTweenConfig().floatProp("BlurStrength", 0f).setEaseType(easeType));
        }
    }

This example is for Radial Blur. To make the others replace the `RadialBlur` member variable and the `"BlurStrength"` property string.

### Stars

This component is called `WarpStarsGroup` and has the same interface as the other 3 (only `Begin()` and `End()`), but internally it's a bit more complicated. I'm using 4 Particle Systems, each of which has a `WarpStars` component that's used by `WarpStarsGroup`. This allows all the Particle Systems to sync up and be controlled in one place.

**WarpStarsGroup**

    public class WarpStarsGroup : MonoBehaviour {

        public List<WarpStars> warpStarSystems;

        public void Begin(float duration, GoEaseType easeType = GoEaseType.Linear)
        {
            foreach(var system in warpStarSystems)
            {
                system.Begin(duration, easeType);
            }
        }

        public void End(float duration, GoEaseType easeType = GoEaseType.Linear)
        {
            foreach (var system in warpStarSystems)
            {
                system.End(duration, easeType);
            }
        }

        public void Stop()
        {
            foreach(var system in warpStarSystems)
            {
                system.Stop();
            }
        }
    }

**WarpStars**

    [RequireComponent(typeof(ParticleSystem))]
    public class WarpStars : MonoBehaviour {

        private float _maxSpeed;
        private ParticleSystem _stars;
        private GoTween _beginTween;

        void Start ()
        {
            _stars = GetComponent<ParticleSystem>();
            var starsMain = _stars.main;
            _maxSpeed = starsMain.startSpeed.constant;
            starsMain.startSpeed = new ParticleSystem.MinMaxCurve(0f);
        }

        public void Begin(float duration, GoEaseType easeType = GoEaseType.Linear)
        {
            var starsMain = _stars.main;
            starsMain.startSpeed = new ParticleSystem.MinMaxCurve(0f);
            _stars.Play();
            _beginTween = Go.to(_stars, duration, new GoTweenConfig().floatProp("startSpeed", _maxSpeed).setEaseType(easeType));
        }

        public void End(float duration, GoEaseType easeType = GoEaseType.Linear)
        {
            if (_beginTween.state == GoTweenState.Running)
            {
                _beginTween.pause();
                _beginTween.destroy();
            }

            Go.to(_stars, duration, new GoTweenConfig().floatProp("startSpeed", 100f).setEaseType(easeType).onComplete(t => Stop()));
        }

        public void Stop()
        {
            _stars.Stop();
        }
    }

## Player

Thanks to the GoKit tweening library, this component is very simple. Before we start the tween we calculate the duration of the voyage based on how far away the destination is and the speed. I've clamped this between 2 and 5 seconds. Any longer than that and the player gets bored. Then we start the tween. The only other things in this component are the methods to set `OnComplete` and `OnUpdate` handlers, which are used in the `WarpManager` to read overall progress on the warp.

    public class WarpPlayer : MonoBehaviour {

        private Action<AbstractGoTween> _onComplete;
        private Action<AbstractGoTween> _onUpdate;

        public void Warp(Vector3 destination, float speed, GoEaseType easeType)
        {
            GameManager.instance.CameraController.ShakeCamera(0.8f, 10, 0.15f);
            var duration = CalculateDuration(destination, speed);
            var tween = Go.to(transform, duration, new GoTweenConfig().vector3Prop("position", destination).setEaseType(easeType).onComplete(_onComplete));
            tween.setOnUpdateHandler(_onUpdate);
        }

        public void SetOnCompleteHandler(Action<AbstractGoTween> handler)
        {
            _onComplete = handler;
        }

        public void SetOnUpdateHandler(Action<AbstractGoTween> handler)
        {
            _onUpdate = handler;
        }

        private float CalculateDuration(Vector3 destination, float speed)
        {
            return Mathf.Clamp(Vector3.Distance(transform.position, destination) / (speed * 100f), 2f, 5f);
        }
    }

## Thrusters

The thrusters are each `GameObject`s with a `Thruster` component. They are all children of a single `GameObject` with a `ThrusterGroup` component. The `Thruster` component has a single method: `AdjustPower(float size)` which, surprisingly, adjusts the power of the thruster's particle system.

The `ThrusterGroup` provides settings for the minimum and maximum power levels of the thrusters through the inspector, and a `SetPower(float level)` method. `SetMaxPower()` is also provided as a convenience method, and that is what's used during warp.

    public class ThrusterGroup : MonoBehaviour {

        public float maxSize = 1.0f;
        public float minSize = 0.2f;
        public List<Thruster> thrusters;

        void Start()
        {
            if (maxSize < minSize) throw new Exception("Max size is smaller than min size");
        }

        public void SetPower(float level)
        {
            var power = (Mathf.Clamp01(level) * (maxSize - minSize)) + minSize;
            foreach(var thruster in thrusters)
            {
                thruster.AdjustPower(power);
            }
        }

        public void SetMaxPower()
        {
            SetPower(1f);
        }
    }

## Camera

The `WarpCamera` component is required to make sure the camera follows the player properly. So before warp starts the camera is positioned behind the player, and then `Warp(Vector3 destination)` is called at the same time as the equivalent method in `WarpPlayer` is called. Both methods perform a tween on their subjects, which results in the player and camera moving at the same speed toward the same destination. But the destination of the camera is slightly offset (backward and upward). This is calculated in the `WarpManager` beforehand.

    public class WarpCamera : MonoBehaviour {

        public void Warp(Vector3 destination, float speed, GoEaseType easeType)
        {
            var duration = CalculateDuration(destination, speed);
            var tween = Go.to(transform, duration, new GoTweenConfig().vector3Prop("position", destination).setEaseType(easeType));
        }

        private float CalculateDuration(Vector3 destination, float speed)
        {
            return Mathf.Clamp(Vector3.Distance(transform.position, destination) / (speed * 100f), 2f, 5f);
        }
    }

