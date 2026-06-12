using UnityEngine;
using System.Collections.Generic;

public class HeadAimAssist : MonoBehaviour
{
    [Header("Aim Settings")]
    public float aimRange = 50f; // Maximum distance to aim
    public float aimSpeed = 10f; // How smoothly it locks on
    public string headTag = "PlayerHead"; // Tag your head colliders with this
    public KeyCode toggleKey = KeyCode.F; // PC toggle key
    public bool aimAssistEnabled = false;

    [Header("References")]
    public Camera mainCam;
    public LayerMask obstacleLayer; // Walls, objects that block vision

    private Transform currentTarget;

    void Update()
    {
        // --- Toggle for PC ---
        if (Input.GetKeyDown(toggleKey))
        {
            aimAssistEnabled = !aimAssistEnabled;
        }

        // Only run if enabled
        if (!aimAssistEnabled) return;

        // Find all possible heads
        List<Transform> allHeads = FindAllHeadsInRange();

        // Pick the closest visible head
        currentTarget = GetClosestVisibleTarget(allHeads);

        // If we have a valid target, smoothly aim at it
        if (currentTarget != null)
        {
            AimAtTarget(currentTarget.position);
        }
    }

    // --- Mobile Toggle Button (attach this to a UI button) ---
    public void ToggleAimAssist()
    {
        aimAssistEnabled = !aimAssistEnabled;
    }

    // Find all heads within range
    List<Transform> FindAllHeadsInRange()
    {
        List<Transform> foundHeads = new List<Transform>();
        GameObject[] heads = GameObject.FindGameObjectsWithTag(headTag);

        foreach (GameObject head in heads)
        {
            float distance = Vector3.Distance(transform.position, head.transform.position);
            if (distance <= aimRange)
            {
                foundHeads.Add(head.transform);
            }
        }
        return foundHeads;
    }

    // Check if head is visible (no walls/obstacles between you and it)
    Transform GetClosestVisibleTarget(List<Transform> targets)
    {
        Transform closest = null;
        float closestDist = Mathf.Infinity;

        foreach (Transform t in targets)
        {
            Vector3 dirToTarget = t.position - transform.position;
            float dist = dirToTarget.magnitude;

            // Raycast to check if anything blocks the view
            if (!Physics.Linecast(transform.position + Vector3.up, t.position, obstacleLayer))
            {
                if (dist < closestDist)
                {
                    closestDist = dist;
                    closest = t;
                }
            }
        }
        return closest;
    }

    // Smoothly rotate camera/weapon toward the target
    void AimAtTarget(Vector3 targetPos)
    {
        Vector3 direction = targetPos - mainCam.transform.position;
        Quaternion targetRotation = Quaternion.LookRotation(direction);
        mainCam.transform.rotation = Quaternion.Lerp(mainCam.transform.rotation, targetRotation, aimSpeed * Time.deltaTime);
    }
}
